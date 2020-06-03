---
title: 读AFNetworking收获与心得
tags: [iOS,AFNetworking,GitHub]
date: 2020-05-26 22:23:00
permalink:
categories: iOS
description: 开发APP，都离不开网络请求，从实习时候开始，很少用系统原生的API去请求数据，而是用iOS著名网络请求框架[AFNetworking](https://github.com/AFNetworking/AFNetworking)，它是对系统网络请求API的高度封装，使用起来相当的简便，接口API，相关类设计非常优秀，非常值得阅读学习。最开始自己也是只会使用，对它进行二次封装，对其源码实现不是很了解，也尝试过阅读，但水平有限，也不能理解其设计思想。最近，在自己工作业余时间，对最新4.0.1源代码进行了阅读，从中还是收获颇多，尝试做一些笔记记录，如有解读描述不对的地方法，希望能批评指正。
image: https://tva1.sinaimg.cn/large/007S8ZIlly1gf8jl77jopj32c80u0wto.jpg
---
<p class="description"></p>

<img src="https://" alt="" style="width:100%" />

<!-- more -->



### 前言

> 开发APP，都离不开网络请求，从实习时候开始，很少用系统原生的API去请求数据，而是用iOS著名网络请求框架[AFNetworking](https://github.com/AFNetworking/AFNetworking)，它是对系统网络请求API的高度封装，使用起来相当的简便，接口API，相关类设计非常优秀，非常值得阅读学习。最开始自己也是只会使用，对它进行二次封装，对其源码实现不是很了解，也尝试过阅读，但水平有限，也不能理解其设计思想。最近，在自己工作业余时间，对最新4.0.1源代码进行了阅读，从中还是收获颇多，尝试做一些笔记记录，如有解读描述不对的地方法，希望能批评指正。

### 整体设计架构

![](https://tva1.sinaimg.cn/large/007S8ZIlly1gf8jl77jopj32c80u0wto.jpg)

### 一、为分类添加属性，关联对象

```objective-c
@interface UIImageView (_AFNetworking)
@property (readwrite, nonatomic, strong, setter = af_setActiveImageDownloadReceipt:) AFImageDownloadReceipt *af_activeImageDownloadReceipt;
@end

@implementation UIImageView (_AFNetworking)

- (AFImageDownloadReceipt *)af_activeImageDownloadReceipt {
    return (AFImageDownloadReceipt *)objc_getAssociatedObject(self, @selector(af_activeImageDownloadReceipt));
}

- (void)af_setActiveImageDownloadReceipt:(AFImageDownloadReceipt *)imageDownloadReceipt {
    objc_setAssociatedObject(self, @selector(af_activeImageDownloadReceipt), imageDownloadReceipt, OBJC_ASSOCIATION_RETAIN_NONATOMIC);
}

@end
```



### 二、并发队列+栅栏函数保证可变容器读写安全

- 创建并发队列

```objc
/// AFAutoPurgingImageCache
NSString *queueName = [NSString stringWithFormat:@"com.alamofire.autopurgingimagecache-%@", [[NSUUID UUID] UUIDString]];
        self.synchronizationQueue = dispatch_queue_create([queueName cStringUsingEncoding:NSASCIIStringEncoding], DISPATCH_QUEUE_CONCURRENT);

```

- dispatch_barrier_async + DISPATCH_QUEUE_CONCURRENT字典存储数据

```objective-c
/// AFAutoPurgingImageCache
- (void)addImage:(UIImage *)image withIdentifier:(NSString *)identifier {
    dispatch_barrier_async(self.synchronizationQueue, ^{
        AFCachedImage *cacheImage = [[AFCachedImage alloc] initWithImage:image identifier:identifier];

        AFCachedImage *previousCachedImage = self.cachedImages[identifier];
        if (previousCachedImage != nil) {
            self.currentMemoryUsage -= previousCachedImage.totalBytes;
        }

        self.cachedImages[identifier] = cacheImage;
        self.currentMemoryUsage += cacheImage.totalBytes;
    });

    dispatch_barrier_async(self.synchronizationQueue, ^{
        if (self.currentMemoryUsage > self.memoryCapacity) {
            UInt64 bytesToPurge = self.currentMemoryUsage - self.preferredMemoryUsageAfterPurge;
            NSMutableArray <AFCachedImage*> *sortedImages = [NSMutableArray arrayWithArray:self.cachedImages.allValues];
            NSSortDescriptor *sortDescriptor = [[NSSortDescriptor alloc] initWithKey:@"lastAccessDate"
                                                                           ascending:YES];
            [sortedImages sortUsingDescriptors:@[sortDescriptor]];

            UInt64 bytesPurged = 0;

            for (AFCachedImage *cachedImage in sortedImages) {
                [self.cachedImages removeObjectForKey:cachedImage.identifier];
                bytesPurged += cachedImage.totalBytes;
                if (bytesPurged >= bytesToPurge) {
                    break;
                }
            }
            self.currentMemoryUsage -= bytesPurged;
        }
    });
}
```

- dispatch_barrier_sync + DISPATCH_QUEUE_CONCURRENT字典移除数据

```objective-c
/// AFAutoPurgingImageCache
- (BOOL)removeImageWithIdentifier:(NSString *)identifier {
    __block BOOL removed = NO;
    dispatch_barrier_sync(self.synchronizationQueue, ^{
        AFCachedImage *cachedImage = self.cachedImages[identifier];
        if (cachedImage != nil) {
            [self.cachedImages removeObjectForKey:identifier];
            self.currentMemoryUsage -= cachedImage.totalBytes;
            removed = YES;
        }
    });
    return removed;
}
```

### 三、图片缓存淘汰算法，通过对图片缓存时间按升序排序，当缓存容量大于设定容量，对图片进行清除，规则越早缓存优先移除，代码如下：

```objective-c
/// AFAutoPurgingImageCache
dispatch_barrier_async(self.synchronizationQueue, ^{
        if (self.currentMemoryUsage > self.memoryCapacity) {
            UInt64 bytesToPurge = self.currentMemoryUsage - self.preferredMemoryUsageAfterPurge;
            NSMutableArray <AFCachedImage*> *sortedImages = [NSMutableArray arrayWithArray:self.cachedImages.allValues];
            NSSortDescriptor *sortDescriptor = [[NSSortDescriptor alloc] initWithKey:@"lastAccessDate"
                                                                           ascending:YES];
            [sortedImages sortUsingDescriptors:@[sortDescriptor]];

            UInt64 bytesPurged = 0;

            for (AFCachedImage *cachedImage in sortedImages) {
                [self.cachedImages removeObjectForKey:cachedImage.identifier];
                bytesPurged += cachedImage.totalBytes;
                if (bytesPurged >= bytesToPurge) {
                    break;
                }
            }
            self.currentMemoryUsage -= bytesPurged;
        }
    });
```

### 四、对于可能出现crash的代码，加@try {} @catch {}，平时用的很少，对保证APP运行安全还是值得使用

```objective-c
/// UIProgressView+AFNetworking
- (void)observeValueForKeyPath:(NSString *)keyPath
                      ofObject:(id)object
                        change:(__unused NSDictionary *)change
                       context:(void *)context
{
    if (context == AFTaskCountOfBytesSentContext || context == AFTaskCountOfBytesReceivedContext) {
        if ([keyPath isEqualToString:NSStringFromSelector(@selector(countOfBytesSent))]) {
            if ([object countOfBytesExpectedToSend] > 0) {
                dispatch_async(dispatch_get_main_queue(), ^{
                    [self setProgress:[object countOfBytesSent] / ([object countOfBytesExpectedToSend] * 1.0f) animated:self.af_uploadProgressAnimated];
                });
            }
        }

        if ([keyPath isEqualToString:NSStringFromSelector(@selector(countOfBytesReceived))]) {
            if ([object countOfBytesExpectedToReceive] > 0) {
                dispatch_async(dispatch_get_main_queue(), ^{
                    [self setProgress:[object countOfBytesReceived] / ([object countOfBytesExpectedToReceive] * 1.0f) animated:self.af_downloadProgressAnimated];
                });
            }
        }

        if ([keyPath isEqualToString:NSStringFromSelector(@selector(state))]) {
            if ([(NSURLSessionTask *)object state] == NSURLSessionTaskStateCompleted) {
                @try {
                    [object removeObserver:self forKeyPath:NSStringFromSelector(@selector(state))];

                    if (context == AFTaskCountOfBytesSentContext) {
                        [object removeObserver:self forKeyPath:NSStringFromSelector(@selector(countOfBytesSent))];
                    }

                    if (context == AFTaskCountOfBytesReceivedContext) {
                        [object removeObserver:self forKeyPath:NSStringFromSelector(@selector(countOfBytesReceived))];
                    }
                }
                @catch (NSException * __unused exception) {}
            }
        }
    }
}
```

### 五、对于一个属性，对外界修改关闭，内部可以修改，可以这样写

- 在.h文件

```objective-c
/// AFNetworkReachabilityManager
/**
 The current network reachability status.
 */
/// 用readonly修饰，保证外界不可修改
@property (readonly, nonatomic, assign) AFNetworkReachabilityStatus networkReachabilityStatus;
```

在.m文件extension中

```objective-c
/// AFNetworkReachabilityManager
/// readwrite修饰，内部可修改其值
@property (readwrite, nonatomic, assign) AFNetworkReachabilityStatus networkReachabilityStatus;
```

### 六、指定方法不可用，并抛出异常

```objective-c
/// AFNetworkReachabilityManager
/**
 *  Unavailable initializer
 */
- (instancetype)init NS_UNAVAILABLE;

- (instancetype)init
{
   /// 抛出异常 
    @throw [NSException exceptionWithName:NSGenericException
                                   reason:@"`-init` unavailable. Use `-initWithReachability:` instead"
                                 userInfo:nil];
    return nil;
}
```

### 七、函数递归调用，将数据处理成想要的结果

```objective-c
/// AFURLRequestSerialization.m

#pragma mark -

FOUNDATION_EXPORT NSArray * AFQueryStringPairsFromDictionary(NSDictionary *dictionary);
FOUNDATION_EXPORT NSArray * AFQueryStringPairsFromKeyAndValue(NSString *key, id value);

NSString * AFQueryStringFromParameters(NSDictionary *parameters) {
    NSMutableArray *mutablePairs = [NSMutableArray array];
    for (AFQueryStringPair *pair in AFQueryStringPairsFromDictionary(parameters)) {
        [mutablePairs addObject:[pair URLEncodedStringValue]];
    }

    return [mutablePairs componentsJoinedByString:@"&"];
}

NSArray * AFQueryStringPairsFromDictionary(NSDictionary *dictionary) {
    return AFQueryStringPairsFromKeyAndValue(nil, dictionary);
}

/// 递归调用次方法，将参数处理成目标类型
NSArray * AFQueryStringPairsFromKeyAndValue(NSString *key, id value) {
    NSMutableArray *mutableQueryStringComponents = [NSMutableArray array];

    NSSortDescriptor *sortDescriptor = [NSSortDescriptor sortDescriptorWithKey:@"description" ascending:YES selector:@selector(compare:)];

    if ([value isKindOfClass:[NSDictionary class]]) {
        NSDictionary *dictionary = value;
        // Sort dictionary keys to ensure consistent ordering in query string, which is important when deserializing potentially ambiguous sequences, such as an array of dictionaries
        for (id nestedKey in [dictionary.allKeys sortedArrayUsingDescriptors:@[ sortDescriptor ]]) {
            id nestedValue = dictionary[nestedKey];
            if (nestedValue) {
                [mutableQueryStringComponents addObjectsFromArray:AFQueryStringPairsFromKeyAndValue((key ? [NSString stringWithFormat:@"%@[%@]", key, nestedKey] : nestedKey), nestedValue)];
            }
        }
    } else if ([value isKindOfClass:[NSArray class]]) {
        NSArray *array = value;
        for (id nestedValue in array) {
            [mutableQueryStringComponents addObjectsFromArray:AFQueryStringPairsFromKeyAndValue([NSString stringWithFormat:@"%@[]", key], nestedValue)];
        }
    } else if ([value isKindOfClass:[NSSet class]]) {
        NSSet *set = value;
        for (id obj in [set sortedArrayUsingDescriptors:@[ sortDescriptor ]]) {
            [mutableQueryStringComponents addObjectsFromArray:AFQueryStringPairsFromKeyAndValue(key, obj)];
        }
    } else {
        [mutableQueryStringComponents addObject:[[AFQueryStringPair alloc] initWithField:key value:value]];
    }

    return mutableQueryStringComponents;
}
```

### 八、手动触发KVO

- 对于要手动触发的属性，进行特殊处理

```objective-c
+ (BOOL)automaticallyNotifiesObserversForKey:(NSString *)key {
    if ([AFHTTPRequestSerializerObservedKeyPaths() containsObject:key]) {
        return NO;
    }

    return [super automaticallyNotifiesObserversForKey:key];
}
```

- 手动触发KVO

```objective-c
- (void)setAllowsCellularAccess:(BOOL)allowsCellularAccess {
    [self willChangeValueForKey:NSStringFromSelector(@selector(allowsCellularAccess))];
    _allowsCellularAccess = allowsCellularAccess;
    [self didChangeValueForKey:NSStringFromSelector(@selector(allowsCellularAccess))];
}

- (void)setCachePolicy:(NSURLRequestCachePolicy)cachePolicy {
    [self willChangeValueForKey:NSStringFromSelector(@selector(cachePolicy))];
    _cachePolicy = cachePolicy;
    [self didChangeValueForKey:NSStringFromSelector(@selector(cachePolicy))];
}

- (void)setHTTPShouldHandleCookies:(BOOL)HTTPShouldHandleCookies {
    [self willChangeValueForKey:NSStringFromSelector(@selector(HTTPShouldHandleCookies))];
    _HTTPShouldHandleCookies = HTTPShouldHandleCookies;
    [self didChangeValueForKey:NSStringFromSelector(@selector(HTTPShouldHandleCookies))];
}

- (void)setHTTPShouldUsePipelining:(BOOL)HTTPShouldUsePipelining {
    [self willChangeValueForKey:NSStringFromSelector(@selector(HTTPShouldUsePipelining))];
    _HTTPShouldUsePipelining = HTTPShouldUsePipelining;
    [self didChangeValueForKey:NSStringFromSelector(@selector(HTTPShouldUsePipelining))];
}

- (void)setNetworkServiceType:(NSURLRequestNetworkServiceType)networkServiceType {
    [self willChangeValueForKey:NSStringFromSelector(@selector(networkServiceType))];
    _networkServiceType = networkServiceType;
    [self didChangeValueForKey:NSStringFromSelector(@selector(networkServiceType))];
}

- (void)setTimeoutInterval:(NSTimeInterval)timeoutInterval {
    [self willChangeValueForKey:NSStringFromSelector(@selector(timeoutInterval))];
    _timeoutInterval = timeoutInterval;
    [self didChangeValueForKey:NSStringFromSelector(@selector(timeoutInterval))];
}

```

### 九、实现NSCopying协议

- 遵循NSCopying协议

```objective-c
/// AFURLRequestSerialization
@protocol AFURLRequestSerialization <NSObject, NSSecureCoding, NSCopying>

/**
 Returns a request with the specified parameters encoded into a copy of the original request.

 @param request The original request.
 @param parameters The parameters to be encoded.
 @param error The error that occurred while attempting to encode the request parameters.

 @return A serialized request.
 */
- (nullable NSURLRequest *)requestBySerializingRequest:(NSURLRequest *)request
                               withParameters:(nullable id)parameters
                                        error:(NSError * _Nullable __autoreleasing *)error NS_SWIFT_NOTHROW;

@end
```

- 实现`- (instancetype)copyWithZone:(NSZone *)zone`方法

  ```objective-c
  /// AFURLRequestSerialization.m
  - (instancetype)copyWithZone:(NSZone *)zone {
      AFHTTPRequestSerializer *serializer = [[[self class] allocWithZone:zone] init];
      dispatch_sync(self.requestHeaderModificationQueue, ^{
          serializer.mutableHTTPRequestHeaders = [self.mutableHTTPRequestHeaders mutableCopyWithZone:zone];
      });
      serializer.queryStringSerializationStyle = self.queryStringSerializationStyle;
      serializer.queryStringSerialization = self.queryStringSerialization;
  
      return serializer;
  }
  ```

### 十、NSSecureCoding协议

- 遵循NSSecureCoding协议

```objective-c
/// AFSecurityPolicy
@interface AFSecurityPolicy : NSObject <NSSecureCoding, NSCopying>
```

- 实现

```objective-c
/// AFSecurityPolicy
#pragma mark - NSSecureCoding

+ (BOOL)supportsSecureCoding {
    return YES;
}

/// decode
- (instancetype)initWithCoder:(NSCoder *)decoder {

    self = [self init];
    if (!self) {
        return nil;
    }

    self.SSLPinningMode = [[decoder decodeObjectOfClass:[NSNumber class] forKey:NSStringFromSelector(@selector(SSLPinningMode))] unsignedIntegerValue];
    self.allowInvalidCertificates = [decoder decodeBoolForKey:NSStringFromSelector(@selector(allowInvalidCertificates))];
    self.validatesDomainName = [decoder decodeBoolForKey:NSStringFromSelector(@selector(validatesDomainName))];
    self.pinnedCertificates = [decoder decodeObjectOfClass:[NSSet class] forKey:NSStringFromSelector(@selector(pinnedCertificates))];

    return self;
}

/// encode
- (void)encodeWithCoder:(NSCoder *)coder {
    [coder encodeObject:[NSNumber numberWithUnsignedInteger:self.SSLPinningMode] forKey:NSStringFromSelector(@selector(SSLPinningMode))];
    [coder encodeBool:self.allowInvalidCertificates forKey:NSStringFromSelector(@selector(allowInvalidCertificates))];
    [coder encodeBool:self.validatesDomainName forKey:NSStringFromSelector(@selector(validatesDomainName))];
    [coder encodeObject:self.pinnedCertificates forKey:NSStringFromSelector(@selector(pinnedCertificates))];
}
```

### 结语

> 这也是自己第一次阅读源码写的记录博客，当自己对作者源码思想有所理解时候，内心也是非常开心与满足，阅读优秀开源库源码，对自己编程的架构思想，代码设计能力还是非常有帮助的。由于自己水平有限，有解读不对地方，还请不吝指正，我会立即做出修改，后续有新的体会与收获，也会在这篇博客中更新。

<hr />
