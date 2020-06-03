---
title: MJExtension源码阅读笔记
tags: [iOS,JSON解析,MJExtension,开源库]
date: 2020-05-29 21:11:48
permalink:
categories: iOS
description: 编写native界面，就避免不了和JSON数据打交道，界面数据填充，我们可以通过原生的字典取值，这样做似乎不是很优雅，于是我们通过一个字典去初始化一个模型类，通过属性名取值，但这样似乎不是很自动化。于是后来就有了JSON自动转模型框架，如MJExtension、YYModel等高性能转化框架。MJExtension可以轻松实现JSON和模型互转，自定义别名，自定义转换，归档解档，总之相当的强大。
image: https://tva1.sinaimg.cn/large/007S8ZIlly1gfai6ntgzjj322w0u047h.jpg
---
<p class="description"></p>

<!-- more -->

## 前言

> 编写native界面，就避免不了和JSON数据打交道，界面数据填充，我们可以通过原生的字典取值，这样做似乎不是很优雅，于是我们通过一个字典去初始化一个模型类，通过属性名取值，但这样似乎不是很自动化。于是后来就有了JSON自动转模型框架，如MJExtension、YYModel等高性能转化框架。MJExtension可以轻松实现JSON和模型互转，自定义别名，自定义转换，归档解档，总之相当的强大。

## 实现原理浅析

通过阅读源码，其实现原理主要运用了Runtime技术、KVC实现的。

## 框架思维导图

![](https://tva1.sinaimg.cn/large/007S8ZIlly1gfai6ntgzjj322w0u047h.jpg)

## 实用技巧

### 标注方法过期

当我们设计一个开源库的时候，有时候考虑的可能并不是很全面，比如方法命名不准确，不能表明用途，或者不推荐使用了，可以给出相应的提示

```objective-c
/// MJExtensionConst

/// 过期
#define MJExtensionDeprecated(instead) NS_DEPRECATED(2_0, 2_0, 2_0, 2_0, instead)

/// 具体使用
- (void)mj_keyValuesDidFinishConvertingToObject MJExtensionDeprecated("请使用`mj_didConvertToObjectWithKeyValues:`替代");
```

### 遍历Protocol的PropertyList

```objective-c
+ (BOOL)isFromNSObjectProtocolProperty:(NSString *)propertyName
{
    if (!propertyName) return NO;
    
    static NSSet<NSString *> *objectProtocolPropertyNames;
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        unsigned int count = 0;
        objc_property_t *propertyList = protocol_copyPropertyList(@protocol(NSObject), &count);
        NSMutableSet *propertyNames = [NSMutableSet setWithCapacity:count];
        for (int i = 0; i < count; i++) {
            objc_property_t property = propertyList[i];
            NSString *propertyName = [NSString stringWithCString:property_getName(property) encoding:NSUTF8StringEncoding];
            if (propertyName) {
                [propertyNames addObject:propertyName];
            }
        }
        objectProtocolPropertyNames = [propertyNames copy];
        free(propertyList);
    });
    
    return [objectProtocolPropertyNames containsObject:propertyName];
}

```

### 获得类所有成员变量

```objective-c
/// NSObject+MJKeyValue
+ (NSMutableArray *)mj_properties
{
    NSMutableArray *cachedProperties = [self mj_propertyDictForKey:&MJCachedPropertiesKey][NSStringFromClass(self)];
    if (cachedProperties == nil) {
    
        if (cachedProperties == nil) {
            cachedProperties = [NSMutableArray array];
            
            /// 遍历类
            [self mj_enumerateClasses:^(__unsafe_unretained Class c, BOOL *stop) {
                // 1.获得所有的成员变量
                unsigned int outCount = 0;
                objc_property_t *properties = class_copyPropertyList(c, &outCount);
                
                // 2.遍历每一个成员变量
                for (unsigned int i = 0; i<outCount; i++) {
                    MJProperty *property = [MJProperty cachedPropertyWithProperty:properties[i]];
                    // 过滤掉Foundation框架类里面的属性
                    if ([MJFoundation isClassFromFoundation:property.srcClass]) continue;
                    // 过滤掉`hash`, `superclass`, `description`, `debugDescription`
                    if ([MJFoundation isFromNSObjectProtocolProperty:property.name]) continue;
                    
                    property.srcClass = c;
                    [property setOriginKey:[self mj_propertyKey:property.name] forClass:self];
                    [property setObjectClassInArray:[self mj_propertyObjectClassInArray:property.name] forClass:self];
                    [cachedProperties addObject:property];
                }
                
                // 3.释放内存
                free(properties);
            }];
            
            [self mj_propertyDictForKey:&MJCachedPropertiesKey][NSStringFromClass(self)] = cachedProperties;
        }
    }
    
    return cachedProperties;
}
```



### [属性类型说明](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/ObjCRuntimeGuide/Articles/ocrtPropertyIntrospection.html#//apple_ref/doc/uid/TP40008048-CH101-SW1)

Objective-C type encodings

| Code               | Meaning                                                      |
| :----------------- | :----------------------------------------------------------- |
| `c`                | A `char`                                                     |
| `i`                | An `int`                                                     |
| `s`                | A `short`                                                    |
| `l`                | A `long``l` is treated as a 32-bit quantity on 64-bit programs. |
| `q`                | A `long long`                                                |
| `C`                | An `unsigned char`                                           |
| `I`                | An `unsigned int`                                            |
| `S`                | An `unsigned short`                                          |
| `L`                | An `unsigned long`                                           |
| `Q`                | An `unsigned long long`                                      |
| `f`                | A `float`                                                    |
| `d`                | A `double`                                                   |
| `B`                | A C++ `bool` or a C99 `_Bool`                                |
| `v`                | A `void`                                                     |
| `*`                | A character string (`char *`)                                |
| `@`                | An object (whether statically typed or typed `id`)           |
| `#`                | A class object (`Class`)                                     |
| `:`                | A method selector (`SEL`)                                    |
| [*array type*]     | An array                                                     |
| {*name=type...*}   | A structure                                                  |
| (*name*=*type...*) | A union                                                      |
| `b`num             | A bit field of *num* bits                                    |
| `^`type            | A pointer to *type*                                          |
| `?`                | An unknown type (among other things, this code is used for function pointers) |

### 对于可变字典或数组，添加信号量锁保证线程安全

```objective-c
/// MJProperty
_propertyKeysLock = dispatch_semaphore_create(1);

- (NSArray *)propertyKeysForClass:(Class)c
{
    NSString *key = NSStringFromClass(c);
    if (!key) return nil;
    
    MJ_LOCK(self.propertyKeysLock);
    NSArray *propertyKeys = self.propertyKeysDict[key];
    MJ_UNLOCK(self.propertyKeysLock);
    return propertyKeys;
}

```

### 通过block实现类遍历

类似系统数组Block遍历可，可通过`stop`控制遍历结束

```objective-c
/**
 *  遍历所有类的block（父类）
 */
typedef void (^MJClassesEnumeration)(Class c, BOOL *stop);

+ (void)mj_enumerateClasses:(MJClassesEnumeration)enumeration
{
    // 1.没有block就直接返回
    if (enumeration == nil) return;
    
    // 2.停止遍历的标记
    BOOL stop = NO;
    
    // 3.当前正在遍历的类
    Class c = self;
    
    // 4.开始遍历每一个类
    while (c && !stop) {
        // 4.1.执行操作
        enumeration(c, &stop);
        
        // 4.2.获得父类
        c = class_getSuperclass(c);
        
        if ([MJFoundation isClassFromFoundation:c]) break;
    }
}
```

### 基于Runtime自动归档解档

- 如果模型属性很多的话，手动实现每个属性的归档解档，还是相当麻烦的，通过Runtime遍历成员变量，调用KVC，实现自动化归档解档

```objective-c
/// NSObject (MJCoding)
@implementation NSObject (MJCoding)

- (void)mj_encode:(NSCoder *)encoder
{
    Class clazz = [self class];
    
    NSArray *allowedCodingPropertyNames = [clazz mj_totalAllowedCodingPropertyNames];
    NSArray *ignoredCodingPropertyNames = [clazz mj_totalIgnoredCodingPropertyNames];
    
    [clazz mj_enumerateProperties:^(MJProperty *property, BOOL *stop) {
        // 检测是否被忽略
        if (allowedCodingPropertyNames.count && ![allowedCodingPropertyNames containsObject:property.name]) return;
        if ([ignoredCodingPropertyNames containsObject:property.name]) return;
        
        id value = [property valueForObject:self];
        if (value == nil) return;
        [encoder encodeObject:value forKey:property.name];
    }];
}

- (void)mj_decode:(NSCoder *)decoder
{
    Class clazz = [self class];
    
    NSArray *allowedCodingPropertyNames = [clazz mj_totalAllowedCodingPropertyNames];
    NSArray *ignoredCodingPropertyNames = [clazz mj_totalIgnoredCodingPropertyNames];
    
    [clazz mj_enumerateProperties:^(MJProperty *property, BOOL *stop) {
        // 检测是否被忽略
        if (allowedCodingPropertyNames.count && ![allowedCodingPropertyNames containsObject:property.name]) return;
        if ([ignoredCodingPropertyNames containsObject:property.name]) return;
        
        id value = [decoder decodeObjectForKey:property.name];
        if (value == nil) { // 兼容以前的MJExtension版本
            value = [decoder decodeObjectForKey:[@"_" stringByAppendingString:property.name]];
        }
        if (value == nil) return;
        [property setValue:value forObject:self];
    }];
}
@end
```

- 具体使用，在.h文件遵循`NSCoding`协议，在.m添加`MJExtensionCodingImplementation`宏定义即可

```objective-c
@implementation xxx
  
MJExtensionCodingImplementation
  
@end
```

- `NSCoding`从源码可以看出只实现了`NSCoding`编码解码，如果想使用更安全的编码解码，可以遵循`NSSecureCoding`

```objective-c
// Objects which are safe to be encoded and decoded across privilege boundaries should adopt NSSecureCoding instead of NSCoding. Secure coders (those that respond YES to requiresSecureCoding) will only encode objects that adopt the NSSecureCoding protocol.
// NOTE: NSSecureCoding guarantees only that an archive contains the classes it claims. It makes no guarantees about the suitability for consumption by the receiver of the decoded content of the archive. Archived objects which  may trigger code evaluation should be validated independently by the consumer of the objects to verify that no malicious code is executed (i.e. by checking key paths, selectors etc. specified in the archive).

@protocol NSSecureCoding <NSCoding>
@required
// This property must return YES on all classes that allow secure coding. Subclasses of classes that adopt NSSecureCoding and override initWithCoder: must also override this method and return YES.
// The Secure Coding Guide should be consulted when writing methods that decode data.
@property (class, readonly) BOOL supportsSecureCoding;
@end
```

- 在.m文件重写getter方法

```objective-c
+ (BOOL)supportsSecureCoding {
    return YES;
}
```

## 常用方法

- 自定义属性别名

```objective-c
/**
 *  将属性名换为其他key去字典中取值
 *
 *  @return 字典中的key是属性名，value是从字典中取值用的key
 */
+ (NSDictionary *)mj_replacedKeyFromPropertyName;
```

- 模型包含数组模型

```objective-c
/**
 *  数组中需要转换的模型类
 *
 *  @return 字典中的key是数组属性名，value是数组中存放模型的Class（Class类型或者NSString类型）
 */
+ (NSDictionary *)mj_objectClassInArray;
```

- 字典转模型

```objective-c
/**
 *  通过字典来创建一个模型
 *  @param keyValues 字典(可以是NSDictionary、NSData、NSString)
 *  @return 新建的对象
 */
+ (instancetype)mj_objectWithKeyValues:(id)keyValues;
```

- 模型转字典

```objective-c
/**
 *  将模型转成字典
 *  @return 字典
 */
- (NSMutableDictionary *)mj_keyValues;
```

- 当字典转模型完毕时调用

```objective-c
- (void)mj_didConvertToObjectWithKeyValues:(NSDictionary *)keyValues;
```

- 自定义转换,返回新的值

```objective-c
- (id)mj_newValueFromOldValue:(id)oldValue property:(MJProperty *)property;
```

## 结语

> 对MJExtension源码阅读了一遍，从中还是收获颇多。作者优秀的框架设计能力，每个类都有其单一的功能，通过组合统一，实现了一套友好易于使用的API接口，上手非常方便。从中也了解了Runtime技术的强大，应用也是非常的广泛。由于自己技术水平有限，对于作者高度封装的一些方法，现在理解还是有些吃力，对于runtime技术在项目中的使用，也并不是很多。以后有时间，还会重新阅读，更新新的体会与心得。对于开源库的阅读，更多的是要理解作者的设计思想，底层实现原理，希望自己能从开源库的阅读，能够学习优秀编程思想，经验，提升自己编码水平、风格、和健壮性。最后，文章如有描述不对的地方，希望不吝批评指教。

<hr />
