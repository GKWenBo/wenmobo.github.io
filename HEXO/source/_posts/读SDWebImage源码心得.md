---
title: 读SDWebImage源码心得
tags: [iOS,SDWebImage]
date: 2020-06-07 15:17:52
permalink:
categories: iOS
description: 笔者在业余时间对iOS著名图片加载框架[SDWebImage ](https://github.com/SDWebImage/SDWebImage)**V5.8.0**源码进行了阅读，发现和V5.0之前版本架构设计有很大的不同，V5.0之后的版本是面向协议的架构设计，将图片缓存、编码解码、下载、自定义转换定义了一套标准协议方法，使用者想自定义一些操作，通过遵循协议，可以很方便的进行扩展。总之，SDWebImage源码是非常值得阅读，它应用了软件设计原则，和设计模式，使软件架构层次清晰，更容易理解阅读，扩展。SDWebImage源码还是有点多的，在零零散散的时间花了好几天粗略的读了一遍，收获还是颇多的，由于笔者现在水平有限，对于有些源码理解不是很到位，有理解描述不对的地方，希望能批评指正。
image: https://tva1.sinaimg.cn/large/007S8ZIlly1gfjr9lek4rj31110dk0uo.jpg
---
<p class="description"></p>

<img src="https://tva1.sinaimg.cn/large/007S8ZIlly1gfjr9lek4rj31110dk0uo.jpg" alt="" style="width:100%" />

<!-- more -->

## 一、前言

> 笔者在业余时间对iOS著名图片加载框架[SDWebImage ](https://github.com/SDWebImage/SDWebImage)**V5.8.0**源码进行了阅读，发现和V5.0之前版本架构设计有很大的不同，V5.0之后的版本是面向协议的架构设计，将图片缓存、编码解码、下载、自定义转换定义了一套标准协议方法，使用者想自定义一些操作，通过遵循协议，可以很方便的进行扩展。总之，SDWebImage源码是非常值得阅读，它应用了软件设计原则，和设计模式，使软件架构层次清晰，更容易理解阅读，扩展。SDWebImage源码还是有点多的，在零零散散的时间花了几天粗略的读了一遍，收获还是颇多的，由于笔者现在水平有限，对于有些源码理解不是很到位，有理解描述不对的地方，希望能批评指正。

## 二、官方架构图解

### High Level Diagram

![](https://tva1.sinaimg.cn/large/007S8ZIlly1gfjr6cmcxxj30sg0lcjtb.jpg)

### Overall Class Diagram

![](https://tva1.sinaimg.cn/large/007S8ZIlly1gfjr7pni02j30zo0u0gwu.jpg)

### Top Level API Diagram

![](https://tva1.sinaimg.cn/large/007S8ZIlly1gfjr8tfo67j31ac0hn438.jpg)

### Main Sequence Diagram

![](https://tva1.sinaimg.cn/large/007S8ZIlly1gfjr9lek4rj31110dk0uo.jpg)

## 三、思维导图

![](https://tva1.sinaimg.cn/large/007S8ZIlly1gfjrxs5kkfj30u00xb4lh.jpg)

### 3.1、Utils

主要定义了一些常用工具类，宏定义（如，平台判断，信号量锁）、不同平台命名统一、位移枚举，错误枚举，加载菊花，图片加载动画等。

### 3.2、Private

私有工具类，文件、颜色，自定义NSOperation封装，封装NSBezierPath等

- 封装了定时器SDDisplayLink及避免循环引用SDWeakProxy

### 3.3、Cache

图片缓存类，处理图片缓存，查找，删除，最大存储容量配置，过期数据处理。

- SDImageCacheConfig：缓存配置类，实现了NSCopying协议，图片默认磁盘缓存为一周

- SDMemoryCache：内存缓存，继承NSCache实现，其内部用到了NSMapTable，默认shouldUseWeakMemoryCache为YES

  ```objective-c
  /// 定义属性
  @property (nonatomic, strong, nonnull) NSMapTable<KeyType, ObjectType> *weakCache; // strong-weak cache
  
  /// 创建
  self.weakCache = [[NSMapTable alloc] initWithKeyOptions:NSPointerFunctionsStrongMemory valueOptions:NSPointerFunctionsWeakMemory capacity:0];
  ```

- SDDiskCache：磁盘缓存，实现了存储、查询、删除等同步/异步操作，图片容量超容处理

  ```objective-c
  /// APP进入后台，对过期数据进行处理
  #if SD_UIKIT
  - (void)applicationDidEnterBackground:(NSNotification *)notification {
      if (!self.config.shouldRemoveExpiredDataWhenEnterBackground) {
          return;
      }
      Class UIApplicationClass = NSClassFromString(@"UIApplication");
      if(!UIApplicationClass || ![UIApplicationClass respondsToSelector:@selector(sharedApplication)]) {
          return;
      }
      UIApplication *application = [UIApplication performSelector:@selector(sharedApplication)];
      __block UIBackgroundTaskIdentifier bgTask = [application beginBackgroundTaskWithExpirationHandler:^{
          // Clean up any unfinished task business by marking where you
          // stopped or ending the task outright.
          [application endBackgroundTask:bgTask];
          bgTask = UIBackgroundTaskInvalid;
      }];
  
      // Start the long-running task and return immediately.
      [self deleteOldFilesWithCompletionBlock:^{
          [application endBackgroundTask:bgTask];
          bgTask = UIBackgroundTaskInvalid;
      }];
  }
  #endif
  ```

  清理核心方法

  ```objective-c
  - (void)removeExpiredData {
      NSURL *diskCacheURL = [NSURL fileURLWithPath:self.diskCachePath isDirectory:YES];
      
      // Compute content date key to be used for tests
      NSURLResourceKey cacheContentDateKey = NSURLContentModificationDateKey;
      switch (self.config.diskCacheExpireType) {
          case SDImageCacheConfigExpireTypeAccessDate:
              cacheContentDateKey = NSURLContentAccessDateKey;
              break;
          case SDImageCacheConfigExpireTypeModificationDate:
              cacheContentDateKey = NSURLContentModificationDateKey;
              break;
          case SDImageCacheConfigExpireTypeCreationDate:
              cacheContentDateKey = NSURLCreationDateKey;
              break;
          case SDImageCacheConfigExpireTypeChangeDate:
              cacheContentDateKey = NSURLAttributeModificationDateKey;
              break;
          default:
              break;
      }
      
      NSArray<NSString *> *resourceKeys = @[NSURLIsDirectoryKey, cacheContentDateKey, NSURLTotalFileAllocatedSizeKey];
      
      // This enumerator prefetches useful properties for our cache files.
      NSDirectoryEnumerator *fileEnumerator = [self.fileManager enumeratorAtURL:diskCacheURL
                                                 includingPropertiesForKeys:resourceKeys
                                                                    options:NSDirectoryEnumerationSkipsHiddenFiles
                                                               errorHandler:NULL];
      
      NSDate *expirationDate = (self.config.maxDiskAge < 0) ? nil: [NSDate dateWithTimeIntervalSinceNow:-self.config.maxDiskAge];
      NSMutableDictionary<NSURL *, NSDictionary<NSString *, id> *> *cacheFiles = [NSMutableDictionary dictionary];
      NSUInteger currentCacheSize = 0;
      
      // Enumerate all of the files in the cache directory.  This loop has two purposes:
      //
      //  1. Removing files that are older than the expiration date.
      //  2. Storing file attributes for the size-based cleanup pass.
      NSMutableArray<NSURL *> *urlsToDelete = [[NSMutableArray alloc] init];
      for (NSURL *fileURL in fileEnumerator) {
          NSError *error;
          NSDictionary<NSString *, id> *resourceValues = [fileURL resourceValuesForKeys:resourceKeys error:&error];
          
          // Skip directories and errors.
          if (error || !resourceValues || [resourceValues[NSURLIsDirectoryKey] boolValue]) {
              continue;
          }
          
          // Remove files that are older than the expiration date;
          NSDate *modifiedDate = resourceValues[cacheContentDateKey];
          if (expirationDate && [[modifiedDate laterDate:expirationDate] isEqualToDate:expirationDate]) {
              [urlsToDelete addObject:fileURL];
              continue;
          }
          
          // Store a reference to this file and account for its total size.
          NSNumber *totalAllocatedSize = resourceValues[NSURLTotalFileAllocatedSizeKey];
          currentCacheSize += totalAllocatedSize.unsignedIntegerValue;
          cacheFiles[fileURL] = resourceValues;
      }
      
      for (NSURL *fileURL in urlsToDelete) {
          [self.fileManager removeItemAtURL:fileURL error:nil];
      }
      
      // If our remaining disk cache exceeds a configured maximum size, perform a second
      // size-based cleanup pass.  We delete the oldest files first.
      NSUInteger maxDiskSize = self.config.maxDiskSize;
      if (maxDiskSize > 0 && currentCacheSize > maxDiskSize) {
          // Target half of our maximum cache size for this cleanup pass.
          const NSUInteger desiredCacheSize = maxDiskSize / 2;
          
          // Sort the remaining cache files by their last modification time or last access time (oldest first).
          NSArray<NSURL *> *sortedFiles = [cacheFiles keysSortedByValueWithOptions:NSSortConcurrent
                                                                   usingComparator:^NSComparisonResult(id obj1, id obj2) {
                                                                       return [obj1[cacheContentDateKey] compare:obj2[cacheContentDateKey]];
                                                                   }];
          
          // Delete files until we fall below our desired cache size.
          for (NSURL *fileURL in sortedFiles) {
              if ([self.fileManager removeItemAtURL:fileURL error:nil]) {
                  NSDictionary<NSString *, id> *resourceValues = cacheFiles[fileURL];
                  NSNumber *totalAllocatedSize = resourceValues[NSURLTotalFileAllocatedSizeKey];
                  currentCacheSize -= totalAllocatedSize.unsignedIntegerValue;
                  
                  if (currentCacheSize < desiredCacheSize) {
                      break;
                  }
              }
          }
      }
  }
  ```

- SDImageCachesManager：管理遵循SDImageCache协议的对象，同时自己也遵循SDImageCache协议，将相关操作派发的具体的处理类，如磁盘、内存缓存。

### 3.4、Prefetcher

> Prefetch some URLs in the cache for future use. Images are downloaded in low priority.

预取缓存中的一些url以备将来使用。以低优先级下载图像。

### 3.5、Transformer

> /**
>
>  A transformer protocol to transform the image load from cache or from download.
>
>  You can provide transformer to cache and manager (Through the `transformer` property or context option `SDWebImageContextImageTransformer`).
>
>  
>
>  @note The transform process is called from a global queue in order to not to block the main queue.
>
>  */

图片转化相关属性包装，现在提供了这几种**SDImagePipelineTransformer**，**SDImageRoundCornerTransformer**、**SDImageResizingTransformer**、**SDImageCroppingTransformer**、**SDImageFlippingTransformer**、**SDImageRotationTransformer**、**SDImageTintTransformer**、**SDImageBlurTransformer**、**SDImageFilterTransformer**，见名知意这些Transformer的用途，其实现类**UIImage+Transform** Category中。

### 3.6、Downloader

图片下载处理相关类，下载结果通过Block回调。提供了根据URL下载图片，可控制暂停、取消，最大下载并发数量，默认是6个，下载一些状态获取。

**SDWebImageDownloaderConfig**：下载相关配置，如最大并发数量，请求超时设置，队列执行顺序，证书设置等

```objective-c
/**
 The class contains all the config for image downloader
 @note This class conform to NSCopying, make sure to add the property in `copyWithZone:` as well.
 */
@interface SDWebImageDownloaderConfig : NSObject <NSCopying>

/**
 Gets the default downloader config used for shared instance or initialization when it does not provide any downloader config. Such as `SDWebImageDownloader.sharedDownloader`.
 @note You can modify the property on default downloader config, which can be used for later created downloader instance. The already created downloader instance does not get affected.
 */
@property (nonatomic, class, readonly, nonnull) SDWebImageDownloaderConfig *defaultDownloaderConfig;

/**
 * The maximum number of concurrent downloads.
 * Defaults to 6.
 */
@property (nonatomic, assign) NSInteger maxConcurrentDownloads;

/**
 * The timeout value (in seconds) for each download operation.
 * Defaults to 15.0.
 */
@property (nonatomic, assign) NSTimeInterval downloadTimeout;

/**
 * The minimum interval about progress percent during network downloading. Which means the next progress callback and current progress callback's progress percent difference should be larger or equal to this value. However, the final finish download progress callback does not get effected.
 * The value should be 0.0-1.0.
 * @note If you're using progressive decoding feature, this will also effect the image refresh rate.
 * @note This value may enhance the performance if you don't want progress callback too frequently.
 * Defaults to 0, which means each time we receive the new data from URLSession, we callback the progressBlock immediately.
 */
@property (nonatomic, assign) double minimumProgressInterval;

/**
 * The custom session configuration in use by NSURLSession. If you don't provide one, we will use `defaultSessionConfiguration` instead.
 * Defatuls to nil.
 * @note This property does not support dynamic changes, means it's immutable after the downloader instance initialized.
 */
@property (nonatomic, strong, nullable) NSURLSessionConfiguration *sessionConfiguration;

/**
 * Gets/Sets a subclass of `SDWebImageDownloaderOperation` as the default
 * `NSOperation` to be used each time SDWebImage constructs a request
 * operation to download an image.
 * Defaults to nil.
 * @note Passing `NSOperation<SDWebImageDownloaderOperation>` to set as default. Passing `nil` will revert to `SDWebImageDownloaderOperation`.
 */
@property (nonatomic, assign, nullable) Class operationClass;

/**
 * Changes download operations execution order.
 * Defaults to `SDWebImageDownloaderFIFOExecutionOrder`.
 */
@property (nonatomic, assign) SDWebImageDownloaderExecutionOrder executionOrder;

/**
 * Set the default URL credential to be set for request operations.
 * Defaults to nil.
 */
@property (nonatomic, copy, nullable) NSURLCredential *urlCredential;

/**
 * Set username using for HTTP Basic authentication.
 * Defaults to nil.
 */
@property (nonatomic, copy, nullable) NSString *username;

/**
 * Set password using for HTTP Basic authentication.
 * Defautls to nil.
 */
@property (nonatomic, copy, nullable) NSString *password;

@end

```

- **SDWebImageDownloaderRequestModifier**：封装NSMutableURLRequest相关设置，可通过提供Block配置。
- **SDWebImageDownloaderResponseModifier**：请求NSHTTPURLResponse配置，可通过提供Block配置。
- **SDWebImageDownloaderDecryptor**：图片二进制加密处理，可通过提供Block配置。默认Base64。

**SDImageLoader**：定义了图片下载方法协议**SDImageLoader**，图片解码方法

```objective-c
/**
 This is the built-in decoding process for image download from network or local file.
 @note If you want to implement your custom loader with `requestImageWithURL:options:context:progress:completed:` API, but also want to keep compatible with SDWebImage's behavior, you'd better use this to produce image.

 @param imageData The image data from the network. Should not be nil
 @param imageURL The image URL from the input. Should not be nil
 @param options The options arg from the input
 @param context The context arg from the input
 @return The decoded image for current image data load from the network
 */
FOUNDATION_EXPORT UIImage * _Nullable SDImageLoaderDecodeImageData(NSData * _Nonnull imageData, NSURL * _Nonnull imageURL, SDWebImageOptions options, SDWebImageContext * _Nullable context);

/**
 This is the built-in decoding process for image progressive download from network. It's used when `SDWebImageProgressiveLoad` option is set. (It's not required when your loader does not support progressive image loading)
 @note If you want to implement your custom loader with `requestImageWithURL:options:context:progress:completed:` API, but also want to keep compatible with SDWebImage's behavior, you'd better use this to produce image.

 @param imageData The image data from the network so far. Should not be nil
 @param imageURL The image URL from the input. Should not be nil
 @param finished Pass NO to specify the download process has not finished. Pass YES when all image data has finished.
 @param operation The loader operation associated with current progressive download. Why to provide this is because progressive decoding need to store the partial decoded context for each operation to avoid conflict. You should provide the operation from `loadImageWithURL:` method return value.
 @param options The options arg from the input
 @param context The context arg from the input
 @return The decoded progressive image for current image data load from the network
 */
FOUNDATION_EXPORT UIImage * _Nullable SDImageLoaderDecodeProgressiveImageData(NSData * _Nonnull imageData, NSURL * _Nonnull imageURL, BOOL finished,  id<SDWebImageOperation> _Nonnull operation, SDWebImageOptions options, SDWebImageContext * _Nullable context);
```

- **SDWebImageDownloaderOperation**: 自定义NSOperation子类，封装了图片下载任务

  ```objective-c
  /// 图片下载任务方法
  - (void)start {
      @synchronized (self) {
          if (self.isCancelled) {
              self.finished = YES;
              // Operation cancelled by user before sending the request
              [self callCompletionBlocksWithError:[NSError errorWithDomain:SDWebImageErrorDomain code:SDWebImageErrorCancelled userInfo:@{NSLocalizedDescriptionKey : @"Operation cancelled by user before sending the request"}]];
              [self reset];
              return;
          }
  
  #if SD_UIKIT
          Class UIApplicationClass = NSClassFromString(@"UIApplication");
          BOOL hasApplication = UIApplicationClass && [UIApplicationClass respondsToSelector:@selector(sharedApplication)];
          if (hasApplication && [self shouldContinueWhenAppEntersBackground]) {
              __weak typeof(self) wself = self;
              UIApplication * app = [UIApplicationClass performSelector:@selector(sharedApplication)];
              self.backgroundTaskId = [app beginBackgroundTaskWithExpirationHandler:^{
                  [wself cancel];
              }];
          }
  #endif
          NSURLSession *session = self.unownedSession;
          if (!session) {
              NSURLSessionConfiguration *sessionConfig = [NSURLSessionConfiguration defaultSessionConfiguration];
              sessionConfig.timeoutIntervalForRequest = 15;
              
              /**
               *  Create the session for this task
               *  We send nil as delegate queue so that the session creates a serial operation queue for performing all delegate
               *  method calls and completion handler calls.
               */
              session = [NSURLSession sessionWithConfiguration:sessionConfig
                                                      delegate:self
                                                 delegateQueue:nil];
              self.ownedSession = session;
          }
          
          if (self.options & SDWebImageDownloaderIgnoreCachedResponse) {
              // Grab the cached data for later check
              NSURLCache *URLCache = session.configuration.URLCache;
              if (!URLCache) {
                  URLCache = [NSURLCache sharedURLCache];
              }
              NSCachedURLResponse *cachedResponse;
              // NSURLCache's `cachedResponseForRequest:` is not thread-safe, see https://developer.apple.com/documentation/foundation/nsurlcache#2317483
              @synchronized (URLCache) {
                  cachedResponse = [URLCache cachedResponseForRequest:self.request];
              }
              if (cachedResponse) {
                  self.cachedData = cachedResponse.data;
              }
          }
          
          self.dataTask = [session dataTaskWithRequest:self.request];
          self.executing = YES;
      }
  
      if (self.dataTask) {
          if (self.options & SDWebImageDownloaderHighPriority) {
              self.dataTask.priority = NSURLSessionTaskPriorityHigh;
              self.coderQueue.qualityOfService = NSQualityOfServiceUserInteractive;
          } else if (self.options & SDWebImageDownloaderLowPriority) {
              self.dataTask.priority = NSURLSessionTaskPriorityLow;
              self.coderQueue.qualityOfService = NSQualityOfServiceBackground;
          } else {
              self.dataTask.priority = NSURLSessionTaskPriorityDefault;
              self.coderQueue.qualityOfService = NSQualityOfServiceDefault;
          }
          [self.dataTask resume];
          for (SDWebImageDownloaderProgressBlock progressBlock in [self callbacksForKey:kProgressCallbackKey]) {
              progressBlock(0, NSURLResponseUnknownLength, self.request.URL);
          }
          __block typeof(self) strongSelf = self;
          dispatch_async(dispatch_get_main_queue(), ^{
              [[NSNotificationCenter defaultCenter] postNotificationName:SDWebImageDownloadStartNotification object:strongSelf];
          });
      } else {
          [self callCompletionBlocksWithError:[NSError errorWithDomain:SDWebImageErrorDomain code:SDWebImageErrorInvalidDownloadOperation userInfo:@{NSLocalizedDescriptionKey : @"Task can't be initialized"}]];
          [self done];
      }
  }
  ```

  

- **SDWebImageDownloader**：图片下载类，遵循了**SDImageLoader**，内部持有**SDWebImageDownloaderConfig**，**SDWebImageDownloaderResponseModifier**、**SDWebImageDownloaderRequestModifier**等实例类。图片下载核心方法如下：

  ```objective-c
  /// 开始一个图片下载任务
  - (nullable SDWebImageDownloadToken *)downloadImageWithURL:(nullable NSURL *)url
                                                     options:(SDWebImageDownloaderOptions)options
                                                     context:(nullable SDWebImageContext *)context
                                                    progress:(nullable SDWebImageDownloaderProgressBlock)progressBlock
                                                   completed:(nullable SDWebImageDownloaderCompletedBlock)completedBlock {
      // The URL will be used as the key to the callbacks dictionary so it cannot be nil. If it is nil immediately call the completed block with no image or data.
      if (url == nil) {
          if (completedBlock) {
              NSError *error = [NSError errorWithDomain:SDWebImageErrorDomain code:SDWebImageErrorInvalidURL userInfo:@{NSLocalizedDescriptionKey : @"Image url is nil"}];
              completedBlock(nil, nil, error, YES);
          }
          return nil;
      }
      
      SD_LOCK(self.operationsLock);
      id downloadOperationCancelToken;
      NSOperation<SDWebImageDownloaderOperation> *operation = [self.URLOperations objectForKey:url];
      // There is a case that the operation may be marked as finished or cancelled, but not been removed from `self.URLOperations`.
      if (!operation || operation.isFinished || operation.isCancelled) {
          operation = [self createDownloaderOperationWithUrl:url options:options context:context];
          if (!operation) {
              SD_UNLOCK(self.operationsLock);
              if (completedBlock) {
                  NSError *error = [NSError errorWithDomain:SDWebImageErrorDomain code:SDWebImageErrorInvalidDownloadOperation userInfo:@{NSLocalizedDescriptionKey : @"Downloader operation is nil"}];
                  completedBlock(nil, nil, error, YES);
              }
              return nil;
          }
          @weakify(self);
          operation.completionBlock = ^{
              @strongify(self);
              if (!self) {
                  return;
              }
              SD_LOCK(self.operationsLock);
              [self.URLOperations removeObjectForKey:url];
              SD_UNLOCK(self.operationsLock);
          };
          self.URLOperations[url] = operation;
          // Add the handlers before submitting to operation queue, avoid the race condition that operation finished before setting handlers.
          downloadOperationCancelToken = [operation addHandlersForProgress:progressBlock completed:completedBlock];
          // Add operation to operation queue only after all configuration done according to Apple's doc.
          // `addOperation:` does not synchronously execute the `operation.completionBlock` so this will not cause deadlock.
          [self.downloadQueue addOperation:operation];
      } else {
          // When we reuse the download operation to attach more callbacks, there may be thread safe issue because the getter of callbacks may in another queue (decoding queue or delegate queue)
          // So we lock the operation here, and in `SDWebImageDownloaderOperation`, we use `@synchonzied (self)`, to ensure the thread safe between these two classes.
          @synchronized (operation) {
              downloadOperationCancelToken = [operation addHandlersForProgress:progressBlock completed:completedBlock];
          }
          if (!operation.isExecuting) {
              if (options & SDWebImageDownloaderHighPriority) {
                  operation.queuePriority = NSOperationQueuePriorityHigh;
              } else if (options & SDWebImageDownloaderLowPriority) {
                  operation.queuePriority = NSOperationQueuePriorityLow;
              } else {
                  operation.queuePriority = NSOperationQueuePriorityNormal;
              }
          }
      }
      SD_UNLOCK(self.operationsLock);
      
      SDWebImageDownloadToken *token = [[SDWebImageDownloadToken alloc] initWithDownloadOperation:operation];
      token.url = url;
      token.request = operation.request;
      token.downloadOperationCancelToken = downloadOperationCancelToken;
      
      return token;
  }
  
  ```

  - **SDImageLoadersManager**：遵循**SDImageLoader**协议，管理遵循了SDImageLoader协议的数组，主要代码如下：

  ```objective-c
  #pragma mark - SDImageLoader
  
  - (BOOL)canRequestImageForURL:(nullable NSURL *)url {
      NSArray<id<SDImageLoader>> *loaders = self.loaders;
      for (id<SDImageLoader> loader in loaders.reverseObjectEnumerator) {
          if ([loader canRequestImageForURL:url]) {
              return YES;
          }
      }
      return NO;
  }
  
  - (id<SDWebImageOperation>)requestImageWithURL:(NSURL *)url options:(SDWebImageOptions)options context:(SDWebImageContext *)context progress:(SDImageLoaderProgressBlock)progressBlock completed:(SDImageLoaderCompletedBlock)completedBlock {
      if (!url) {
          return nil;
      }
      NSArray<id<SDImageLoader>> *loaders = self.loaders;
      for (id<SDImageLoader> loader in loaders.reverseObjectEnumerator) {
          if ([loader canRequestImageForURL:url]) {
              return [loader requestImageWithURL:url options:options context:context progress:progressBlock completed:completedBlock];
          }
      }
      return nil;
  }
  
  - (BOOL)shouldBlockFailedURLWithURL:(NSURL *)url error:(NSError *)error {
      NSArray<id<SDImageLoader>> *loaders = self.loaders;
      for (id<SDImageLoader> loader in loaders.reverseObjectEnumerator) {
          if ([loader canRequestImageForURL:url]) {
              return [loader shouldBlockFailedURLWithURL:url error:error];
          }
      }
      return NO;
  }
  
  ```

  ### 3.5、Decoder

  图片编/解码。支持的图片格式如下：

  ```objective-c
  /**
   You can use switch case like normal enum. It's also recommended to add a default case. You should not assume anything about the raw value.
   For custom coder plugin, it can also extern the enum for supported format. See `SDImageCoder` for more detailed information.
   */
  typedef NSInteger SDImageFormat NS_TYPED_EXTENSIBLE_ENUM;
  static const SDImageFormat SDImageFormatUndefined = -1;
  static const SDImageFormat SDImageFormatJPEG      = 0;
  static const SDImageFormat SDImageFormatPNG       = 1;
  static const SDImageFormat SDImageFormatGIF       = 2;
  static const SDImageFormat SDImageFormatTIFF      = 3;
  static const SDImageFormat SDImageFormatWebP      = 4; /// SDWebImage默认不支持
  static const SDImageFormat SDImageFormatHEIC      = 5;
  static const SDImageFormat SDImageFormatHEIF      = 6;
  static const SDImageFormat SDImageFormatPDF       = 7;
  static const SDImageFormat SDImageFormatSVG       = 8;
  ```

  对于特殊格式的图片，SDWebImage创建了一个对应的编解码子类，遵循了**SDImageIOAnimatedCoder**，**SDImageIOAnimatedCoder**又遵循了**<SDProgressiveImageCoder, SDAnimatedImageCoder>**协议。这部分内容还是相对来说枯燥复杂的。

  - **SDImageCoder**：定义了主要编解码方法，同时也定义了**SDProgressiveImageCoder**、**SDAnimatedImageCoder**、**SDAnimatedImageProvider**等协议

  ```objective-c
  #pragma mark - Coder
  /**
   This is the image coder protocol to provide custom image decoding/encoding.
   These methods are all required to implement.
   @note Pay attention that these methods are not called from main queue.
   */
  @protocol SDImageCoder <NSObject>
  
  @required
  #pragma mark - Decoding
  /**
   Returns YES if this coder can decode some data. Otherwise, the data should be passed to another coder.
   
   @param data The image data so we can look at it
   @return YES if this coder can decode the data, NO otherwise
   */
  - (BOOL)canDecodeFromData:(nullable NSData *)data;
  
  /**
   Decode the image data to image.
   @note This protocol may supports decode animated image frames. You can use `+[SDImageCoderHelper animatedImageWithFrames:]` to produce an animated image with frames.
  
   @param data The image data to be decoded
   @param options A dictionary containing any decoding options. Pass @{SDImageCoderDecodeScaleFactor: @(1.0)} to specify scale factor for image. Pass @{SDImageCoderDecodeFirstFrameOnly: @(YES)} to decode the first frame only.
   @return The decoded image from data
   */
  - (nullable UIImage *)decodedImageWithData:(nullable NSData *)data
                                     options:(nullable SDImageCoderOptions *)options;
  
  #pragma mark - Encoding
  
  /**
   Returns YES if this coder can encode some image. Otherwise, it should be passed to another coder.
   For custom coder which introduce new image format, you'd better define a new `SDImageFormat` using like this. If you're creating public coder plugin for new image format, also update `https://github.com/rs/SDWebImage/wiki/Coder-Plugin-List` to avoid same value been defined twice.
   * @code
   static const SDImageFormat SDImageFormatHEIF = 10;
   * @endcode
   
   @param format The image format
   @return YES if this coder can encode the image, NO otherwise
   */
  - (BOOL)canEncodeToFormat:(SDImageFormat)format NS_SWIFT_NAME(canEncode(to:));
  
  /**
   Encode the image to image data.
   @note This protocol may supports encode animated image frames. You can use `+[SDImageCoderHelper framesFromAnimatedImage:]` to assemble an animated image with frames.
  
   @param image The image to be encoded
   @param format The image format to encode, you should note `SDImageFormatUndefined` format is also  possible
   @param options A dictionary containing any encoding options. Pass @{SDImageCoderEncodeCompressionQuality: @(1)} to specify compression quality.
   @return The encoded image data
   */
  - (nullable NSData *)encodedDataWithImage:(nullable UIImage *)image
                                     format:(SDImageFormat)format
                                    options:(nullable SDImageCoderOptions *)options;
  
  @end
  
  ```

  - **SDImageCodersManager**：管理了各种图片格式编/解码类，同时遵循了SDImageCoder协议，核心代码如下：

  ```objective-c
  #pragma mark - SDImageCoder
  - (BOOL)canDecodeFromData:(NSData *)data {
      NSArray<id<SDImageCoder>> *coders = self.coders;
      for (id<SDImageCoder> coder in coders.reverseObjectEnumerator) {
          if ([coder canDecodeFromData:data]) {
              return YES;
          }
      }
      return NO;
  }
  
  - (BOOL)canEncodeToFormat:(SDImageFormat)format {
      NSArray<id<SDImageCoder>> *coders = self.coders;
      for (id<SDImageCoder> coder in coders.reverseObjectEnumerator) {
          if ([coder canEncodeToFormat:format]) {
              return YES;
          }
      }
      return NO;
  }
  
  - (UIImage *)decodedImageWithData:(NSData *)data options:(nullable SDImageCoderOptions *)options {
      if (!data) {
          return nil;
      }
      UIImage *image;
      NSArray<id<SDImageCoder>> *coders = self.coders;
      for (id<SDImageCoder> coder in coders.reverseObjectEnumerator) {
          if ([coder canDecodeFromData:data]) {
              image = [coder decodedImageWithData:data options:options];
              break;
          }
      }
      
      return image;
  }
  
  - (NSData *)encodedDataWithImage:(UIImage *)image format:(SDImageFormat)format options:(nullable SDImageCoderOptions *)options {
      if (!image) {
          return nil;
      }
      NSArray<id<SDImageCoder>> *coders = self.coders;
      for (id<SDImageCoder> coder in coders.reverseObjectEnumerator) {
          if ([coder canEncodeToFormat:format]) {
              return [coder encodedDataWithImage:image format:format options:options];
          }
      }
      return nil;
  }
  ```


### 3.7、Manager

UIKit相关UI控件分类API调用上游类， 管理图片缓存、下载、编/解码、转换等操作。

- **SDWebImageManager**：SDWebImage核心类。提供了各种操作实例代码。

初始化方法

```objective-c
/**
 * Allows to specify instance of cache and image loader used with image manager.
 * @return new instance of `SDWebImageManager` with specified cache and loader.
 */
- (nonnull instancetype)initWithCache:(nonnull id<SDImageCache>)cache loader:(nonnull id<SDImageLoader>)loader NS_DESIGNATED_INITIALIZER;
```

下载图片方法定义

```objective-c
/**
 * Downloads the image at the given URL if not present in cache or return the cached version otherwise.
 *
 * @param url            The URL to the image
 * @param options        A mask to specify options to use for this request
 * @param context        A context contains different options to perform specify changes or processes, see `SDWebImageContextOption`. This hold the extra objects which `options` enum can not hold.
 * @param progressBlock  A block called while image is downloading
 *                       @note the progress block is executed on a background queue
 * @param completedBlock A block called when operation has been completed.
 *
 * @return Returns an instance of SDWebImageCombinedOperation, which you can cancel the loading process.
 */
- (nullable SDWebImageCombinedOperation *)loadImageWithURL:(nullable NSURL *)url
                                                   options:(SDWebImageOptions)options
                                                   context:(nullable SDWebImageContext *)context
                                                  progress:(nullable SDImageLoaderProgressBlock)progressBlock
                                                 completed:(nonnull SDInternalCompletionBlock)completedBlock;
```

下载图片方法实现

```objective-c
/// 实现
- (SDWebImageCombinedOperation *)loadImageWithURL:(nullable NSURL *)url
                                          options:(SDWebImageOptions)options
                                          context:(nullable SDWebImageContext *)context
                                         progress:(nullable SDImageLoaderProgressBlock)progressBlock
                                        completed:(nonnull SDInternalCompletionBlock)completedBlock {
    // Invoking this method without a completedBlock is pointless
    NSAssert(completedBlock != nil, @"If you mean to prefetch the image, use -[SDWebImagePrefetcher prefetchURLs] instead");

    // Very common mistake is to send the URL using NSString object instead of NSURL. For some strange reason, Xcode won't
    // throw any warning for this type mismatch. Here we failsafe this error by allowing URLs to be passed as NSString.
    if ([url isKindOfClass:NSString.class]) {
        url = [NSURL URLWithString:(NSString *)url];
    }

    // Prevents app crashing on argument type error like sending NSNull instead of NSURL
    if (![url isKindOfClass:NSURL.class]) {
        url = nil;
    }

    SDWebImageCombinedOperation *operation = [SDWebImageCombinedOperation new];
    operation.manager = self;

    BOOL isFailedUrl = NO;
    if (url) {
        SD_LOCK(self.failedURLsLock);
        isFailedUrl = [self.failedURLs containsObject:url];
        SD_UNLOCK(self.failedURLsLock);
    }

    if (url.absoluteString.length == 0 || (!(options & SDWebImageRetryFailed) && isFailedUrl)) {
        NSString *description = isFailedUrl ? @"Image url is blacklisted" : @"Image url is nil";
        NSInteger code = isFailedUrl ? SDWebImageErrorBlackListed : SDWebImageErrorInvalidURL;
        [self callCompletionBlockForOperation:operation completion:completedBlock error:[NSError errorWithDomain:SDWebImageErrorDomain code:code userInfo:@{NSLocalizedDescriptionKey : description}] url:url];
        return operation;
    }

    SD_LOCK(self.runningOperationsLock);
    [self.runningOperations addObject:operation];
    SD_UNLOCK(self.runningOperationsLock);
    
    // Preprocess the options and context arg to decide the final the result for manager
    SDWebImageOptionsResult *result = [self processedResultForURL:url options:options context:context];
    
    // Start the entry to load image from cache
    [self callCacheProcessForOperation:operation url:url options:result.options context:result.context progress:progressBlock completed:completedBlock];

    return operation;
}


#pragma mark - Private

// Query normal cache process
- (void)callCacheProcessForOperation:(nonnull SDWebImageCombinedOperation *)operation
                                 url:(nonnull NSURL *)url
                             options:(SDWebImageOptions)options
                             context:(nullable SDWebImageContext *)context
                            progress:(nullable SDImageLoaderProgressBlock)progressBlock
                           completed:(nullable SDInternalCompletionBlock)completedBlock {
    // Grab the image cache to use
    id<SDImageCache> imageCache;
    if ([context[SDWebImageContextImageCache] conformsToProtocol:@protocol(SDImageCache)]) {
        imageCache = context[SDWebImageContextImageCache];
    } else {
        imageCache = self.imageCache;
    }
    
    // Get the query cache type
    SDImageCacheType queryCacheType = SDImageCacheTypeAll;
    if (context[SDWebImageContextQueryCacheType]) {
        queryCacheType = [context[SDWebImageContextQueryCacheType] integerValue];
    }
    
    // Check whether we should query cache
    BOOL shouldQueryCache = !SD_OPTIONS_CONTAINS(options, SDWebImageFromLoaderOnly);
    if (shouldQueryCache) {
        NSString *key = [self cacheKeyForURL:url context:context];
        @weakify(operation);
        operation.cacheOperation = [imageCache queryImageForKey:key options:options context:context cacheType:queryCacheType completion:^(UIImage * _Nullable cachedImage, NSData * _Nullable cachedData, SDImageCacheType cacheType) {
            @strongify(operation);
            if (!operation || operation.isCancelled) {
                // Image combined operation cancelled by user
                [self callCompletionBlockForOperation:operation completion:completedBlock error:[NSError errorWithDomain:SDWebImageErrorDomain code:SDWebImageErrorCancelled userInfo:@{NSLocalizedDescriptionKey : @"Operation cancelled by user during querying the cache"}] url:url];
                [self safelyRemoveOperationFromRunning:operation];
                return;
            } else if (context[SDWebImageContextImageTransformer] && !cachedImage) {
                // Have a chance to quary original cache instead of downloading
                [self callOriginalCacheProcessForOperation:operation url:url options:options context:context progress:progressBlock completed:completedBlock];
                return;
            }
            
            // Continue download process
            [self callDownloadProcessForOperation:operation url:url options:options context:context cachedImage:cachedImage cachedData:cachedData cacheType:cacheType progress:progressBlock completed:completedBlock];
        }];
    } else {
        // Continue download process
        [self callDownloadProcessForOperation:operation url:url options:options context:context cachedImage:nil cachedData:nil cacheType:SDImageCacheTypeNone progress:progressBlock completed:completedBlock];
    }
}

// Query original cache process
- (void)callOriginalCacheProcessForOperation:(nonnull SDWebImageCombinedOperation *)operation
                                         url:(nonnull NSURL *)url
                                     options:(SDWebImageOptions)options
                                     context:(nullable SDWebImageContext *)context
                                    progress:(nullable SDImageLoaderProgressBlock)progressBlock
                                   completed:(nullable SDInternalCompletionBlock)completedBlock {
    // Grab the image cache to use
    id<SDImageCache> imageCache;
    if ([context[SDWebImageContextImageCache] conformsToProtocol:@protocol(SDImageCache)]) {
        imageCache = context[SDWebImageContextImageCache];
    } else {
        imageCache = self.imageCache;
    }
    
    // Get the original query cache type
    SDImageCacheType originalQueryCacheType = SDImageCacheTypeNone;
    if (context[SDWebImageContextOriginalQueryCacheType]) {
        originalQueryCacheType = [context[SDWebImageContextOriginalQueryCacheType] integerValue];
    }
    
    // Check whether we should query original cache
    BOOL shouldQueryOriginalCache = (originalQueryCacheType != SDImageCacheTypeNone);
    if (shouldQueryOriginalCache) {
        // Change originContext to mutable
        SDWebImageMutableContext * __block originContext;
        if (context) {
            originContext = [context mutableCopy];
        } else {
            originContext = [NSMutableDictionary dictionary];
        }
        
        // Disable transformer for cache key generation
        id<SDImageTransformer> transformer = originContext[SDWebImageContextImageTransformer];
        originContext[SDWebImageContextImageTransformer] = [NSNull null];
        
        NSString *key = [self cacheKeyForURL:url context:originContext];
        @weakify(operation);
        operation.cacheOperation = [imageCache queryImageForKey:key options:options context:context cacheType:originalQueryCacheType completion:^(UIImage * _Nullable cachedImage, NSData * _Nullable cachedData, SDImageCacheType cacheType) {
            @strongify(operation);
            if (!operation || operation.isCancelled) {
                // Image combined operation cancelled by user
                [self callCompletionBlockForOperation:operation completion:completedBlock error:[NSError errorWithDomain:SDWebImageErrorDomain code:SDWebImageErrorCancelled userInfo:@{NSLocalizedDescriptionKey : @"Operation cancelled by user during querying the cache"}] url:url];
                [self safelyRemoveOperationFromRunning:operation];
                return;
            }
            
            // Add original transformer
            if (transformer) {
                originContext[SDWebImageContextImageTransformer] = transformer;
            }
            
            // Use the store cache process instead of downloading, and ignore .refreshCached option for now
            [self callStoreCacheProcessForOperation:operation url:url options:options context:context downloadedImage:cachedImage downloadedData:cachedData finished:YES progress:progressBlock completed:completedBlock];
            
            [self safelyRemoveOperationFromRunning:operation];
        }];
    } else {
        // Continue download process
        [self callDownloadProcessForOperation:operation url:url options:options context:context cachedImage:nil cachedData:nil cacheType:originalQueryCacheType progress:progressBlock completed:completedBlock];
    }
}

// Download process
- (void)callDownloadProcessForOperation:(nonnull SDWebImageCombinedOperation *)operation
                                    url:(nonnull NSURL *)url
                                options:(SDWebImageOptions)options
                                context:(SDWebImageContext *)context
                            cachedImage:(nullable UIImage *)cachedImage
                             cachedData:(nullable NSData *)cachedData
                              cacheType:(SDImageCacheType)cacheType
                               progress:(nullable SDImageLoaderProgressBlock)progressBlock
                              completed:(nullable SDInternalCompletionBlock)completedBlock {
    // Grab the image loader to use
    id<SDImageLoader> imageLoader;
    if ([context[SDWebImageContextImageLoader] conformsToProtocol:@protocol(SDImageLoader)]) {
        imageLoader = context[SDWebImageContextImageLoader];
    } else {
        imageLoader = self.imageLoader;
    }
    
    // Check whether we should download image from network
    BOOL shouldDownload = !SD_OPTIONS_CONTAINS(options, SDWebImageFromCacheOnly);
    shouldDownload &= (!cachedImage || options & SDWebImageRefreshCached);
    shouldDownload &= (![self.delegate respondsToSelector:@selector(imageManager:shouldDownloadImageForURL:)] || [self.delegate imageManager:self shouldDownloadImageForURL:url]);
    shouldDownload &= [imageLoader canRequestImageForURL:url];
    if (shouldDownload) {
        if (cachedImage && options & SDWebImageRefreshCached) {
            // If image was found in the cache but SDWebImageRefreshCached is provided, notify about the cached image
            // AND try to re-download it in order to let a chance to NSURLCache to refresh it from server.
            [self callCompletionBlockForOperation:operation completion:completedBlock image:cachedImage data:cachedData error:nil cacheType:cacheType finished:YES url:url];
            // Pass the cached image to the image loader. The image loader should check whether the remote image is equal to the cached image.
            SDWebImageMutableContext *mutableContext;
            if (context) {
                mutableContext = [context mutableCopy];
            } else {
                mutableContext = [NSMutableDictionary dictionary];
            }
            mutableContext[SDWebImageContextLoaderCachedImage] = cachedImage;
            context = [mutableContext copy];
        }
        
        @weakify(operation);
        operation.loaderOperation = [imageLoader requestImageWithURL:url options:options context:context progress:progressBlock completed:^(UIImage *downloadedImage, NSData *downloadedData, NSError *error, BOOL finished) {
            @strongify(operation);
            if (!operation || operation.isCancelled) {
                // Image combined operation cancelled by user
                [self callCompletionBlockForOperation:operation completion:completedBlock error:[NSError errorWithDomain:SDWebImageErrorDomain code:SDWebImageErrorCancelled userInfo:@{NSLocalizedDescriptionKey : @"Operation cancelled by user during sending the request"}] url:url];
            } else if (cachedImage && options & SDWebImageRefreshCached && [error.domain isEqualToString:SDWebImageErrorDomain] && error.code == SDWebImageErrorCacheNotModified) {
                // Image refresh hit the NSURLCache cache, do not call the completion block
            } else if ([error.domain isEqualToString:SDWebImageErrorDomain] && error.code == SDWebImageErrorCancelled) {
                // Download operation cancelled by user before sending the request, don't block failed URL
                [self callCompletionBlockForOperation:operation completion:completedBlock error:error url:url];
            } else if (error) {
                [self callCompletionBlockForOperation:operation completion:completedBlock error:error url:url];
                BOOL shouldBlockFailedURL = [self shouldBlockFailedURLWithURL:url error:error options:options context:context];
                
                if (shouldBlockFailedURL) {
                    SD_LOCK(self.failedURLsLock);
                    [self.failedURLs addObject:url];
                    SD_UNLOCK(self.failedURLsLock);
                }
            } else {
                if ((options & SDWebImageRetryFailed)) {
                    SD_LOCK(self.failedURLsLock);
                    [self.failedURLs removeObject:url];
                    SD_UNLOCK(self.failedURLsLock);
                }
                // Continue store cache process
                [self callStoreCacheProcessForOperation:operation url:url options:options context:context downloadedImage:downloadedImage downloadedData:downloadedData finished:finished progress:progressBlock completed:completedBlock];
            }
            
            if (finished) {
                [self safelyRemoveOperationFromRunning:operation];
            }
        }];
    } else if (cachedImage) {
        [self callCompletionBlockForOperation:operation completion:completedBlock image:cachedImage data:cachedData error:nil cacheType:cacheType finished:YES url:url];
        [self safelyRemoveOperationFromRunning:operation];
    } else {
        // Image not in cache and download disallowed by delegate
        [self callCompletionBlockForOperation:operation completion:completedBlock image:nil data:nil error:nil cacheType:SDImageCacheTypeNone finished:YES url:url];
        [self safelyRemoveOperationFromRunning:operation];
    }
}

// Store cache process
- (void)callStoreCacheProcessForOperation:(nonnull SDWebImageCombinedOperation *)operation
                                      url:(nonnull NSURL *)url
                                  options:(SDWebImageOptions)options
                                  context:(SDWebImageContext *)context
                          downloadedImage:(nullable UIImage *)downloadedImage
                           downloadedData:(nullable NSData *)downloadedData
                                 finished:(BOOL)finished
                                 progress:(nullable SDImageLoaderProgressBlock)progressBlock
                                completed:(nullable SDInternalCompletionBlock)completedBlock {
    // the target image store cache type
    SDImageCacheType storeCacheType = SDImageCacheTypeAll;
    if (context[SDWebImageContextStoreCacheType]) {
        storeCacheType = [context[SDWebImageContextStoreCacheType] integerValue];
    }
    // the original store image cache type
    SDImageCacheType originalStoreCacheType = SDImageCacheTypeNone;
    if (context[SDWebImageContextOriginalStoreCacheType]) {
        originalStoreCacheType = [context[SDWebImageContextOriginalStoreCacheType] integerValue];
    }
    // origin cache key
    SDWebImageMutableContext *originContext = [context mutableCopy];
    // disable transformer for cache key generation
    originContext[SDWebImageContextImageTransformer] = [NSNull null];
    NSString *key = [self cacheKeyForURL:url context:originContext];
    id<SDImageTransformer> transformer = context[SDWebImageContextImageTransformer];
    if (![transformer conformsToProtocol:@protocol(SDImageTransformer)]) {
        transformer = nil;
    }
    id<SDWebImageCacheSerializer> cacheSerializer = context[SDWebImageContextCacheSerializer];
    
    BOOL shouldTransformImage = downloadedImage && transformer;
    shouldTransformImage = shouldTransformImage && (!downloadedImage.sd_isAnimated || (options & SDWebImageTransformAnimatedImage));
    shouldTransformImage = shouldTransformImage && (!downloadedImage.sd_isVector || (options & SDWebImageTransformVectorImage));
    BOOL shouldCacheOriginal = downloadedImage && finished;
    
    // if available, store original image to cache
    if (shouldCacheOriginal) {
        // normally use the store cache type, but if target image is transformed, use original store cache type instead
        SDImageCacheType targetStoreCacheType = shouldTransformImage ? originalStoreCacheType : storeCacheType;
        if (cacheSerializer && (targetStoreCacheType == SDImageCacheTypeDisk || targetStoreCacheType == SDImageCacheTypeAll)) {
            dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_HIGH, 0), ^{
                @autoreleasepool {
                    NSData *cacheData = [cacheSerializer cacheDataWithImage:downloadedImage originalData:downloadedData imageURL:url];
                    [self storeImage:downloadedImage imageData:cacheData forKey:key cacheType:targetStoreCacheType options:options context:context completion:^{
                        // Continue transform process
                        [self callTransformProcessForOperation:operation url:url options:options context:context originalImage:downloadedImage originalData:downloadedData finished:finished progress:progressBlock completed:completedBlock];
                    }];
                }
            });
        } else {
            [self storeImage:downloadedImage imageData:downloadedData forKey:key cacheType:targetStoreCacheType options:options context:context completion:^{
                // Continue transform process
                [self callTransformProcessForOperation:operation url:url options:options context:context originalImage:downloadedImage originalData:downloadedData finished:finished progress:progressBlock completed:completedBlock];
            }];
        }
    } else {
        // Continue transform process
        [self callTransformProcessForOperation:operation url:url options:options context:context originalImage:downloadedImage originalData:downloadedData finished:finished progress:progressBlock completed:completedBlock];
    }
}

// Transform process
- (void)callTransformProcessForOperation:(nonnull SDWebImageCombinedOperation *)operation
                                     url:(nonnull NSURL *)url
                                 options:(SDWebImageOptions)options
                                 context:(SDWebImageContext *)context
                           originalImage:(nullable UIImage *)originalImage
                            originalData:(nullable NSData *)originalData
                                finished:(BOOL)finished
                                progress:(nullable SDImageLoaderProgressBlock)progressBlock
                               completed:(nullable SDInternalCompletionBlock)completedBlock {
    // the target image store cache type
    SDImageCacheType storeCacheType = SDImageCacheTypeAll;
    if (context[SDWebImageContextStoreCacheType]) {
        storeCacheType = [context[SDWebImageContextStoreCacheType] integerValue];
    }
    // transformed cache key
    NSString *key = [self cacheKeyForURL:url context:context];
    id<SDImageTransformer> transformer = context[SDWebImageContextImageTransformer];
    if (![transformer conformsToProtocol:@protocol(SDImageTransformer)]) {
        transformer = nil;
    }
    id<SDWebImageCacheSerializer> cacheSerializer = context[SDWebImageContextCacheSerializer];
    
    BOOL shouldTransformImage = originalImage && transformer;
    shouldTransformImage = shouldTransformImage && (!originalImage.sd_isAnimated || (options & SDWebImageTransformAnimatedImage));
    shouldTransformImage = shouldTransformImage && (!originalImage.sd_isVector || (options & SDWebImageTransformVectorImage));
    // if available, store transformed image to cache
    if (shouldTransformImage) {
        dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_HIGH, 0), ^{
            @autoreleasepool {
                UIImage *transformedImage = [transformer transformedImageWithImage:originalImage forKey:key];
                if (transformedImage && finished) {
                    BOOL imageWasTransformed = ![transformedImage isEqual:originalImage];
                    NSData *cacheData;
                    // pass nil if the image was transformed, so we can recalculate the data from the image
                    if (cacheSerializer && (storeCacheType == SDImageCacheTypeDisk || storeCacheType == SDImageCacheTypeAll)) {
                        cacheData = [cacheSerializer cacheDataWithImage:transformedImage originalData:(imageWasTransformed ? nil : originalData) imageURL:url];
                    } else {
                        cacheData = (imageWasTransformed ? nil : originalData);
                    }
                    [self storeImage:transformedImage imageData:cacheData forKey:key cacheType:storeCacheType options:options context:context completion:^{
                        [self callCompletionBlockForOperation:operation completion:completedBlock image:transformedImage data:originalData error:nil cacheType:SDImageCacheTypeNone finished:finished url:url];
                    }];
                } else {
                    [self callCompletionBlockForOperation:operation completion:completedBlock image:transformedImage data:originalData error:nil cacheType:SDImageCacheTypeNone finished:finished url:url];
                }
            }
        });
    } else {
        [self callCompletionBlockForOperation:operation completion:completedBlock image:originalImage data:originalData error:nil cacheType:SDImageCacheTypeNone finished:finished url:url];
    }
}
```

先检查内存是否有缓存，没有从磁盘查找，都没有开启一个图片下载任务，拿到图片数据缓存到内存，磁盘，磁盘查找源码如下：

```objective-c
- (nullable NSOperation *)queryCacheOperationForKey:(nullable NSString *)key options:(SDImageCacheOptions)options context:(nullable SDWebImageContext *)context cacheType:(SDImageCacheType)queryCacheType done:(nullable SDImageCacheQueryCompletionBlock)doneBlock {
    if (!key) {
        if (doneBlock) {
            doneBlock(nil, nil, SDImageCacheTypeNone);
        }
        return nil;
    }
    // Invalid cache type
    if (queryCacheType == SDImageCacheTypeNone) {
        if (doneBlock) {
            doneBlock(nil, nil, SDImageCacheTypeNone);
        }
        return nil;
    }
    
    // First check the in-memory cache...
    UIImage *image;
    if (queryCacheType != SDImageCacheTypeDisk) {
        image = [self imageFromMemoryCacheForKey:key];
    }
    
    if (image) {
        if (options & SDImageCacheDecodeFirstFrameOnly) {
            // Ensure static image
            Class animatedImageClass = image.class;
            if (image.sd_isAnimated || ([animatedImageClass isSubclassOfClass:[UIImage class]] && [animatedImageClass conformsToProtocol:@protocol(SDAnimatedImage)])) {
#if SD_MAC
                image = [[NSImage alloc] initWithCGImage:image.CGImage scale:image.scale orientation:kCGImagePropertyOrientationUp];
#else
                image = [[UIImage alloc] initWithCGImage:image.CGImage scale:image.scale orientation:image.imageOrientation];
#endif
            }
        } else if (options & SDImageCacheMatchAnimatedImageClass) {
            // Check image class matching
            Class animatedImageClass = image.class;
            Class desiredImageClass = context[SDWebImageContextAnimatedImageClass];
            if (desiredImageClass && ![animatedImageClass isSubclassOfClass:desiredImageClass]) {
                image = nil;
            }
        }
    }

    BOOL shouldQueryMemoryOnly = (queryCacheType == SDImageCacheTypeMemory) || (image && !(options & SDImageCacheQueryMemoryData));
    if (shouldQueryMemoryOnly) {
        if (doneBlock) {
            doneBlock(image, nil, SDImageCacheTypeMemory);
        }
        return nil;
    }
    
    // Second check the disk cache...
    NSOperation *operation = [NSOperation new];
    // Check whether we need to synchronously query disk
    // 1. in-memory cache hit & memoryDataSync
    // 2. in-memory cache miss & diskDataSync
    BOOL shouldQueryDiskSync = ((image && options & SDImageCacheQueryMemoryDataSync) ||
                                (!image && options & SDImageCacheQueryDiskDataSync));
    void(^queryDiskBlock)(void) =  ^{
        if (operation.isCancelled) {
            if (doneBlock) {
                doneBlock(nil, nil, SDImageCacheTypeNone);
            }
            return;
        }
        
        @autoreleasepool {
            NSData *diskData = [self diskImageDataBySearchingAllPathsForKey:key];
            UIImage *diskImage;
            SDImageCacheType cacheType = SDImageCacheTypeNone;
            if (image) {
                // the image is from in-memory cache, but need image data
                diskImage = image;
                cacheType = SDImageCacheTypeMemory;
            } else if (diskData) {
                cacheType = SDImageCacheTypeDisk;
                // decode image data only if in-memory cache missed
                diskImage = [self diskImageForKey:key data:diskData options:options context:context];
                if (diskImage && self.config.shouldCacheImagesInMemory) {
                    NSUInteger cost = diskImage.sd_memoryCost;
                    [self.memoryCache setObject:diskImage forKey:key cost:cost];
                }
            }
            
            if (doneBlock) {
                if (shouldQueryDiskSync) {
                    doneBlock(diskImage, diskData, cacheType);
                } else {
                    dispatch_async(dispatch_get_main_queue(), ^{
                        doneBlock(diskImage, diskData, cacheType);
                    });
                }
            }
        }
    };
    
    // Query in ioQueue to keep IO-safe
    if (shouldQueryDiskSync) {
        dispatch_sync(self.ioQueue, queryDiskBlock);
    } else {
        dispatch_async(self.ioQueue, queryDiskBlock);
    }
    
    return operation;
}
```

### 3.8、AnimatedImage

类似[FLAnimatedImage](https://github.com/Flipboard/FLAnimatedImage)高性能播放GIF格式图片，对于大容量GIF图片，直接加载会导致内存暴涨，这点在我负责的APP深有体会，由于GIF图片过大，导致首次加载界面会出现卡顿，界面渲染延迟，后来换成了[FLAnimatedImage](https://github.com/Flipboard/FLAnimatedImage)，解决了播放GIF大图内存的问题。现在SDWebImage也提供了GIT图片加载类，提高性能。

- **SDAnimatedImage**：继承自UIImage
- **SDAnimatedImageView**：继承UIImageView，处理GIF图片播放
- **SDAnimatedImageView+WebCache**：提供类似普通图片加载API，加载网络或本地

```objective-c
/**
 * Set the imageView `image` with an `url`.
 *
 * The download is asynchronous and cached.
 *
 * @param url The url for the image.
 */
- (void)sd_setImageWithURL:(nullable NSURL *)url NS_REFINED_FOR_SWIFT;
```

- **SDAnimatedImagePlayer**：A player to control the playback of animated image, which can be used to drive Animated ImageView or any rendering usage, like CALayer/WatchKit/SwiftUI rendering.

### 3.9、Categories

图片相关分类

- **NSData+ImageContentType**：图片格式判断，字符串转换
- **UIImage+ExtendedCacheData**：给图片关联数据，需遵循NSCoding协议
- **UIImage+GIF**：将二进制数据转化为GIF图片
- **UIImage+Metadata**： UIImage category for image metadata, including animation, loop count, format, incremental, etc.
- **UIImage+MultiFormat**：图片编码解码
- **UIImage+ForceDecode**：UIImage category about force decode feature (avoid Image/IO's lazy decoding during rendering behavior).
- **UIImage+Transform**： Provide some commen method for `UIImage`.Image process is based on Core Graphics and vImage.
- **UIImage+MemoryCacheCost**：图片内存消耗
- **NSImage+Compatibility**：This category is provided to easily write cross-platform(AppKit/UIKit) code. For common usage, see `UIImage+Metadata.h`
- **UIView+WebCacheOperation**：These methods are used to support canceling for UIView image loading, it's designed to be used internal but not external. All the stored operations are weak, so it will be dalloced after image loading finished. If you need to store operations, use your own class to keep a strong reference for them.

### 3.10、WebCache Categories

支持图片设置UI控件设置图片分类。**UIView+WebCache**分类添加了图片主要设置逻辑。其次就是我们熟知的**UIImageView+WebCache**、**UIImageView+HighlightedWebCache**、**UIButton+WebCache**、**NSButton+WebCache**、

- **UIView+WebCache**

```objective-c
/**
 * Set the imageView `image` with an `url` and optionally a placeholder image.
 *
 * The download is asynchronous and cached.
 *
 * @param url            The url for the image.
 * @param placeholder    The image to be set initially, until the image request finishes.
 * @param options        The options to use when downloading the image. @see SDWebImageOptions for the possible values.
 * @param context        A context contains different options to perform specify changes or processes, see `SDWebImageContextOption`. This hold the extra objects which `options` enum can not hold.
 * @param setImageBlock  Block used for custom set image code. If not provide, use the built-in set image code (supports `UIImageView/NSImageView` and `UIButton/NSButton` currently)
 * @param progressBlock  A block called while image is downloading
 *                       @note the progress block is executed on a background queue
 * @param completedBlock A block called when operation has been completed.
 *   This block has no return value and takes the requested UIImage as first parameter and the NSData representation as second parameter.
 *   In case of error the image parameter is nil and the third parameter may contain an NSError.
 *
 *   The forth parameter is an `SDImageCacheType` enum indicating if the image was retrieved from the local cache
 *   or from the memory cache or from the network.
 *
 *   The fith parameter normally is always YES. However, if you provide SDWebImageAvoidAutoSetImage with SDWebImageProgressiveLoad options to enable progressive downloading and set the image yourself. This block is thus called repeatedly with a partial image. When image is fully downloaded, the
 *   block is called a last time with the full image and the last parameter set to YES.
 *
 *   The last parameter is the original image URL
 */
- (void)sd_internalSetImageWithURL:(nullable NSURL *)url
                  placeholderImage:(nullable UIImage *)placeholder
                           options:(SDWebImageOptions)options
                           context:(nullable SDWebImageContext *)context
                     setImageBlock:(nullable SDSetImageBlock)setImageBlock
                          progress:(nullable SDImageLoaderProgressBlock)progressBlock
                         completed:(nullable SDInternalCompletionBlock)completedBlock;
/**
 * Cancel the current image load
 */
- (void)sd_cancelCurrentImageLoad;

#if SD_UIKIT || SD_MAC

#pragma mark - Image Transition

/**
 The image transition when image load finished. See `SDWebImageTransition`.
 If you specify nil, do not do transition. Defautls to nil.
 */
@property (nonatomic, strong, nullable) SDWebImageTransition *sd_imageTransition;

#pragma mark - Image Indicator

/**
 The image indicator during the image loading. If you do not need indicator, specify nil. Defaults to nil
 The setter will remove the old indicator view and add new indicator view to current view's subview.
 @note Because this is UI related, you should access only from the main queue.
 */
@property (nonatomic, strong, nullable) id<SDWebImageIndicator> sd_imageIndicator;

#endif
```

方法实现：

```objective-c
- (void)sd_internalSetImageWithURL:(nullable NSURL *)url
                  placeholderImage:(nullable UIImage *)placeholder
                           options:(SDWebImageOptions)options
                           context:(nullable SDWebImageContext *)context
                     setImageBlock:(nullable SDSetImageBlock)setImageBlock
                          progress:(nullable SDImageLoaderProgressBlock)progressBlock
                         completed:(nullable SDInternalCompletionBlock)completedBlock {
    if (context) {
        // copy to avoid mutable object
        context = [context copy];
    } else {
        context = [NSDictionary dictionary];
    }
    NSString *validOperationKey = context[SDWebImageContextSetImageOperationKey];
    if (!validOperationKey) {
        // pass through the operation key to downstream, which can used for tracing operation or image view class
        validOperationKey = NSStringFromClass([self class]);
        SDWebImageMutableContext *mutableContext = [context mutableCopy];
        mutableContext[SDWebImageContextSetImageOperationKey] = validOperationKey;
        context = [mutableContext copy];
    }
    self.sd_latestOperationKey = validOperationKey;
    /// 取消上一次的下载任务
    [self sd_cancelImageLoadOperationWithKey:validOperationKey];
    self.sd_imageURL = url;
    
    /// 如果不延迟加载展位图，就先显示占位图
    if (!(options & SDWebImageDelayPlaceholder)) {
        dispatch_main_async_safe(^{
            [self sd_setImage:placeholder imageData:nil basedOnClassOrViaCustomSetImageBlock:setImageBlock cacheType:SDImageCacheTypeNone imageURL:url];
        });
    }
    
    if (url) {
        // reset the progress
        NSProgress *imageProgress = objc_getAssociatedObject(self, @selector(sd_imageProgress));
        if (imageProgress) {
            imageProgress.totalUnitCount = 0;
            imageProgress.completedUnitCount = 0;
        }
        
#if SD_UIKIT || SD_MAC
        // check and start image indicator
        [self sd_startImageIndicator];
        id<SDWebImageIndicator> imageIndicator = self.sd_imageIndicator;
#endif
        SDWebImageManager *manager = context[SDWebImageContextCustomManager];
        if (!manager) {
            manager = [SDWebImageManager sharedManager];
        } else {
            // remove this manager to avoid retain cycle (manger -> loader -> operation -> context -> manager)
            SDWebImageMutableContext *mutableContext = [context mutableCopy];
            mutableContext[SDWebImageContextCustomManager] = nil;
            context = [mutableContext copy];
        }
        
        SDImageLoaderProgressBlock combinedProgressBlock = ^(NSInteger receivedSize, NSInteger expectedSize, NSURL * _Nullable targetURL) {
            if (imageProgress) {
                imageProgress.totalUnitCount = expectedSize;
                imageProgress.completedUnitCount = receivedSize;
            }
#if SD_UIKIT || SD_MAC
            if ([imageIndicator respondsToSelector:@selector(updateIndicatorProgress:)]) {
                double progress = 0;
                if (expectedSize != 0) {
                    progress = (double)receivedSize / expectedSize;
                }
                progress = MAX(MIN(progress, 1), 0); // 0.0 - 1.0
                dispatch_async(dispatch_get_main_queue(), ^{
                    [imageIndicator updateIndicatorProgress:progress];
                });
            }
#endif
          /// 回调下载进度  
          if (progressBlock) {
                progressBlock(receivedSize, expectedSize, targetURL);
            }
        };
        @weakify(self);
        id <SDWebImageOperation> operation = [manager loadImageWithURL:url options:options context:context progress:combinedProgressBlock completed:^(UIImage *image, NSData *data, NSError *error, SDImageCacheType cacheType, BOOL finished, NSURL *imageURL) {
            @strongify(self);
            if (!self) { return; }
            // if the progress not been updated, mark it to complete state
            if (imageProgress && finished && !error && imageProgress.totalUnitCount == 0 && imageProgress.completedUnitCount == 0) {
                imageProgress.totalUnitCount = SDWebImageProgressUnitCountUnknown;
                imageProgress.completedUnitCount = SDWebImageProgressUnitCountUnknown;
            }
            
#if SD_UIKIT || SD_MAC
            // check and stop image indicator
            if (finished) {
                [self sd_stopImageIndicator];
            }
#endif
            
            BOOL shouldCallCompletedBlock = finished || (options & SDWebImageAvoidAutoSetImage);
            BOOL shouldNotSetImage = ((image && (options & SDWebImageAvoidAutoSetImage)) ||
                                      (!image && !(options & SDWebImageDelayPlaceholder)));
            SDWebImageNoParamsBlock callCompletedBlockClojure = ^{
                if (!self) { return; }
                if (!shouldNotSetImage) {
                    [self sd_setNeedsLayout];
                }
                if (completedBlock && shouldCallCompletedBlock) {
                    completedBlock(image, data, error, cacheType, finished, url);
                }
            };
            
            // case 1a: we got an image, but the SDWebImageAvoidAutoSetImage flag is set
            // OR
            // case 1b: we got no image and the SDWebImageDelayPlaceholder is not set
            if (shouldNotSetImage) {
                dispatch_main_async_safe(callCompletedBlockClojure);
                return;
            }
            
            UIImage *targetImage = nil;
            NSData *targetData = nil;
            if (image) {
                // case 2a: we got an image and the SDWebImageAvoidAutoSetImage is not set
                targetImage = image;
                targetData = data;
            } else if (options & SDWebImageDelayPlaceholder) {
                // case 2b: we got no image and the SDWebImageDelayPlaceholder flag is set
                targetImage = placeholder;
                targetData = nil;
            }
            
#if SD_UIKIT || SD_MAC
            // check whether we should use the image transition
            SDWebImageTransition *transition = nil;
            if (finished && (options & SDWebImageForceTransition || cacheType == SDImageCacheTypeNone)) {
                transition = self.sd_imageTransition;
            }
#endif
            dispatch_main_async_safe(^{
#if SD_UIKIT || SD_MAC
                /// 设置缓存或网络加载图片
                [self sd_setImage:targetImage imageData:targetData basedOnClassOrViaCustomSetImageBlock:setImageBlock transition:transition cacheType:cacheType imageURL:imageURL];
#else
                [self sd_setImage:targetImage imageData:targetData basedOnClassOrViaCustomSetImageBlock:setImageBlock cacheType:cacheType imageURL:imageURL];
#endif
                callCompletedBlockClojure();
            });
        }];
        
        /// 存储图片下载operation
        [self sd_setImageLoadOperation:operation forKey:validOperationKey];
    } else {
#if SD_UIKIT || SD_MAC
        [self sd_stopImageIndicator];
#endif
        dispatch_main_async_safe(^{
            if (completedBlock) {
                NSError *error = [NSError errorWithDomain:SDWebImageErrorDomain code:SDWebImageErrorInvalidURL userInfo:@{NSLocalizedDescriptionKey : @"Image url is nil"}];
                completedBlock(nil, nil, error, SDImageCacheTypeNone, YES, url);
            }
        });
    }
}

- (void)sd_setImage:(UIImage *)image imageData:(NSData *)imageData basedOnClassOrViaCustomSetImageBlock:(SDSetImageBlock)setImageBlock cacheType:(SDImageCacheType)cacheType imageURL:(NSURL *)imageURL {
#if SD_UIKIT || SD_MAC
    [self sd_setImage:image imageData:imageData basedOnClassOrViaCustomSetImageBlock:setImageBlock transition:nil cacheType:cacheType imageURL:imageURL];
#else
    // watchOS does not support view transition. Simplify the logic
    if (setImageBlock) {
        setImageBlock(image, imageData, cacheType, imageURL);
    } else if ([self isKindOfClass:[UIImageView class]]) {
        UIImageView *imageView = (UIImageView *)self;
        [imageView setImage:image];
    }
#endif
}

#if SD_UIKIT || SD_MAC
- (void)sd_setImage:(UIImage *)image imageData:(NSData *)imageData basedOnClassOrViaCustomSetImageBlock:(SDSetImageBlock)setImageBlock transition:(SDWebImageTransition *)transition cacheType:(SDImageCacheType)cacheType imageURL:(NSURL *)imageURL {
    UIView *view = self;
    SDSetImageBlock finalSetImageBlock;
    if (setImageBlock) {
        finalSetImageBlock = setImageBlock;
    } else if ([view isKindOfClass:[UIImageView class]]) {
        UIImageView *imageView = (UIImageView *)view;
        finalSetImageBlock = ^(UIImage *setImage, NSData *setImageData, SDImageCacheType setCacheType, NSURL *setImageURL) {
            imageView.image = setImage;
        };
    }
#if SD_UIKIT
    else if ([view isKindOfClass:[UIButton class]]) {
        UIButton *button = (UIButton *)view;
        finalSetImageBlock = ^(UIImage *setImage, NSData *setImageData, SDImageCacheType setCacheType, NSURL *setImageURL) {
            [button setImage:setImage forState:UIControlStateNormal];
        };
    }
#endif
#if SD_MAC
    else if ([view isKindOfClass:[NSButton class]]) {
        NSButton *button = (NSButton *)view;
        finalSetImageBlock = ^(UIImage *setImage, NSData *setImageData, SDImageCacheType setCacheType, NSURL *setImageURL) {
            button.image = setImage;
        };
    }
#endif
    
    if (transition) {
#if SD_UIKIT
        [UIView transitionWithView:view duration:0 options:0 animations:^{
            // 0 duration to let UIKit render placeholder and prepares block
            if (transition.prepares) {
                transition.prepares(view, image, imageData, cacheType, imageURL);
            }
        } completion:^(BOOL finished) {
            [UIView transitionWithView:view duration:transition.duration options:transition.animationOptions animations:^{
                if (finalSetImageBlock && !transition.avoidAutoSetImage) {
                    finalSetImageBlock(image, imageData, cacheType, imageURL);
                }
                if (transition.animations) {
                    transition.animations(view, image);
                }
            } completion:transition.completion];
        }];
#elif SD_MAC
        [NSAnimationContext runAnimationGroup:^(NSAnimationContext * _Nonnull prepareContext) {
            // 0 duration to let AppKit render placeholder and prepares block
            prepareContext.duration = 0;
            if (transition.prepares) {
                transition.prepares(view, image, imageData, cacheType, imageURL);
            }
        } completionHandler:^{
            [NSAnimationContext runAnimationGroup:^(NSAnimationContext * _Nonnull context) {
                context.duration = transition.duration;
                #pragma clang diagnostic push
                #pragma clang diagnostic ignored "-Wdeprecated-declarations"
                CAMediaTimingFunction *timingFunction = transition.timingFunction;
                #pragma clang diagnostic pop
                if (!timingFunction) {
                    timingFunction = SDTimingFunctionFromAnimationOptions(transition.animationOptions);
                }
                context.timingFunction = timingFunction;
                context.allowsImplicitAnimation = SD_OPTIONS_CONTAINS(transition.animationOptions, SDWebImageAnimationOptionAllowsImplicitAnimation);
                if (finalSetImageBlock && !transition.avoidAutoSetImage) {
                    finalSetImageBlock(image, imageData, cacheType, imageURL);
                }
                CATransition *trans = SDTransitionFromAnimationOptions(transition.animationOptions);
                if (trans) {
                    [view.layer addAnimation:trans forKey:kCATransition];
                }
                if (transition.animations) {
                    transition.animations(view, image);
                }
            } completionHandler:^{
                if (transition.completion) {
                    transition.completion(YES);
                }
            }];
        }];
#endif
    } else {
        if (finalSetImageBlock) {
            finalSetImageBlock(image, imageData, cacheType, imageURL);
        }
    }
}
#endif
```

取消下载任务

```objective-c
- (void)sd_cancelImageLoadOperationWithKey:(nullable NSString *)key {
    if (key) {
        // Cancel in progress downloader from queue
        SDOperationsDictionary *operationDictionary = [self sd_operationDictionary];
        id<SDWebImageOperation> operation;
        
        @synchronized (self) {
            operation = [operationDictionary objectForKey:key];
        }
        if (operation) {
            if ([operation conformsToProtocol:@protocol(SDWebImageOperation)]) {
                [operation cancel];
            }
            @synchronized (self) {
                [operationDictionary removeObjectForKey:key];
            }
        }
    }
}

```

总的来说，5.0之后的版本，图片设置分类代码简化了许多，因为相关逻辑都被抽离到了Manager里，分类里之需关注方法调用，不用再处理复杂图片的查询，下载，缓存等逻辑。

### 3.11、SDWebImageMapKit

主要为MKAnnotationView添加了设置图片分类。

## 四、软件架构设计需知识储备

![](https://tva1.sinaimg.cn/large/007S8ZIlly1gfjtj8emsuj311p0u0478.jpg)

对源码阅读一遍，我所理解的设计图片加载库所需要的知识储备

> - CocoTouch各种framework
> - 多线程
> - 图片编/解码
> - 网络
> - 文件系统
> - 跨平台适配
> - 设计模式
> - 软件设计原则
> - 算法

## 五、相关插件

### Coders for additional image formats

|                            Plugin                            |                         Description                          |
| :----------------------------------------------------------: | :----------------------------------------------------------: |
| [SDWebImageWebPCoder](https://github.com/SDWebImage/SDWebImageWebPCoder) | coder for WebP format. Based on [libwebp](https://chromium.googlesource.com/webm/libwebp) |
| [SDWebImageHEIFCoder](https://github.com/SDWebImage/SDWebImageHEIFCoder) | coder for HEIF format, iOS 8+/macOS 10.10+ support. Based on [libheif](https://github.com/strukturag/libheif) |
| [SDWebImageBPGCoder](https://github.com/SDWebImage/SDWebImageBPGCoder) | coder for BPG format. Based on [libbpg](https://github.com/mirrorer/libbpg) |
| [SDWebImageFLIFCoder](https://github.com/SDWebImage/SDWebImageFLIFCoder) | coder for FLIF format. Based on [libflif](https://github.com/FLIF-hub/FLIF) |
| [SDWebImageAVIFCoder](https://github.com/SDWebImage/SDWebImageAVIFCoder) | coder for AVIF (AV1-based) format. Based on [libavif](https://github.com/AOMediaCodec/libavif) |
| [SDWebImagePDFCoder](https://github.com/SDWebImage/SDWebImagePDFCoder) |    coder for PDF vector format. Using built-in frameworks    |
| [SDWebImageSVGCoder](https://github.com/SDWebImage/SDWebImageSVGCoder) |    coder for SVG vector format. Using built-in frameworks    |
| [SDWebImageLottieCoder](https://github.com/SDWebImage/SDWebImageLottieCoder) | coder for Lottie animation format. Based on [rlottie](https://github.com/Samsung/rlottie) |

### Custom Caches

|                            Plugin                            |                         Description                          |
| :----------------------------------------------------------: | :----------------------------------------------------------: |
| [SDWebImageYYPlugin](https://github.com/SDWebImage/SDWebImageYYPlugin) | plugin to support caching images with [YYCache](https://github.com/ibireme/YYCache) |
| [SDWebImagePINPlugin](https://github.com/SDWebImage/SDWebImagePINPlugin) | plugin to support caching images with [PINCache](https://github.com/pinterest/PINCache) |

### Custom Loaders

|                            Plugin                            |                         Description                          |
| :----------------------------------------------------------: | :----------------------------------------------------------: |
| [SDWebImagePhotosPlugin](https://github.com/SDWebImage/SDWebImagePhotosPlugin) | plugin to support loading images from Photos (using `Photos.framework`) |
| [SDWebImageLinkPlugin](https://github.com/SDWebImage/SDWebImageLinkPlugin) | plugin to support loading images from rich link url, as well as `LPLinkView` (using `LinkPresentation.framework`) |

### Integration with 3rd party libraries

|                            Plugin                            |                         Description                          |
| :----------------------------------------------------------: | :----------------------------------------------------------: |
| [SDWebImageLottiePlugin](https://github.com/SDWebImage/SDWebImageLottiePlugin) | plugin to support [Lottie-iOS](https://github.com/airbnb/lottie-ios), vector animation rending with remote JSON files |
| [SDWebImageSVGKitPlugin](https://github.com/SDWebImage/SDWebImageLottiePlugin) | plugin to support [SVGKit](https://github.com/SVGKit/SVGKit), SVG rendering using Core Animation, iOS 8+/macOS 10.10+ support |
| [SDWebImageFLPlugin](https://github.com/SDWebImage/SDWebImageFLPlugin) | plugin to support [FLAnimatedImage](https://github.com/Flipboard/FLAnimatedImage) as the engine for animated GIFs |
| [SDWebImageYYPlugin](https://github.com/SDWebImage/SDWebImageYYPlugin) | plugin to integrate [YYImage](https://github.com/ibireme/YYImage) & [YYCache](https://github.com/ibireme/YYCache) for image rendering & caching |

## 六、结语

> 到这里，对源码的解读也就告一段落了。最新版本的架构设计也更加合理，使用协议，使扩展相当灵活，也更容易插件化，对各个功能能够轻松实现替换。框架图片编码解码处理，对于现在的我来说，理解还是有些吃力，总的来说，这次的源码阅读，我收获颇多，好的框架，更易于让人使用，阅读，理解，希望自己在以后编码设计中，能够汲取其优秀的编程设计思想，灵活运用。以后有时间，也会继续阅读源码，相信每次阅读，都会有新的体会与收获。如文章有解读不对的地方，希望能批评指正。

<hr />
