---
title: iOS-WBCategories
tags: [iOS,Objective-C]
date: 2018-07-08 17:26:13
permalink:
categories: iOS
description: 在日常开发过程中，我们也经常用到分类。分类（Category）是OC中的特有语法，它是表示一个指向分类的结构体的指针。原则上它只能增加方法，不能增加成员（实例）变量
image:
---
<p class="description"></p>

<!--more-->

### 更新日志
> 2018-06-19 整理文章目录结构，更新常用分类方法
### 一、官方介绍
> You use categories to define additional methods of an existing class—even one whose source code is unavailable to you—without subclassing. You typically use a category to add methods to an existing class, such as one defined in the Cocoa frameworks. The added methods are inherited by subclasses and are indistinguishable at runtime from the original methods of the class. You can also use categories of your own classes to:

> *   Distribute the implementation of your own classes into separate source files—for example, you could group the methods of a large class into several categories and put each category in a different file.
> *   Declare private methods.

> You add methods to a class by declaring them in an interface file under a category name and defining them in an implementation file under the same name. The category name indicates that the methods are an extension to a class declared elsewhere, not a new class.

### 二、[WBCategories](https://github.com/wenmobo/WBCategories)
在我们日常开发中，通常我们会把项目中常用一些公用的方法抽离出来，写在分类文件当中，**WBCategories**这个工程就是我平时开发过程中常用整理的分类，分类文件主要包含系统框架和宏定义如下：
> - **UIKit**
> - **Foundation**
> - **AVFoundation**
> - **CoreTelephony**
> - **QuartzCore**
> - **常用Macro**

最开始这些分类文件也挺零散的，大部分分类方法都是在网上查询资料，也参考了一些大神在Github上开源库，有些分类方法也是自己写的。下面我会介绍上面框架下常用的一些分类方法。
### 三、Foundation
- 计算文字size，手动布局用需计算文字size
```
- (CGSize)wb_sizeForFont:(UIFont *)font
                    size:(CGSize)size
                    mode:(NSLineBreakMode)lineBreakMode {
    CGSize result;
    if (!font) font = [UIFont systemFontOfSize:12];
    if ([self respondsToSelector:@selector(boundingRectWithSize:options:attributes:context:)]) {
        NSMutableDictionary *attr = [NSMutableDictionary new];
        attr[NSFontAttributeName] = font;
        if (lineBreakMode != NSLineBreakByWordWrapping) {
            NSMutableParagraphStyle *paragraphStyle = [NSMutableParagraphStyle new];
            paragraphStyle.lineBreakMode = lineBreakMode;
            attr[NSParagraphStyleAttributeName] = paragraphStyle;
        }
        CGRect rect = [self boundingRectWithSize:size
                                         options:NSStringDrawingUsesLineFragmentOrigin | NSStringDrawingUsesFontLeading
                                      attributes:attr context:nil];
        result = rect.size;
    } else {
#pragma clang diagnostic push
#pragma clang diagnostic ignored "-Wdeprecated-declarations"
        result = [self sizeWithFont:font constrainedToSize:size lineBreakMode:lineBreakMode];
#pragma clang diagnostic pop
    }
    return result;
}
```
- 字典，数组，可变字符串防越界，主要是通过Runtime实现，在数据越界的情况下APP不会闪退奔溃，还能正常运行，由于篇幅的关系只贴数组的实现方法
```
 + (void)load {
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        [NSObject swizzleInstanceMethodWithOriginSel:@selector(objectAtIndex:)
                                         swizzledSel:@selector(_wb_safe_ZeroObjectAtIndex:)
                                           selfClass:NSClassFromString(@"__NSArray0")];
        [NSObject swizzleInstanceMethodWithOriginSel:@selector(objectAtIndex:)
                                         swizzledSel:@selector(_wb_safe_singleObjectAtIndex:)
                                           selfClass:NSClassFromString(@"__NSSingleObjectArrayI")];
        [NSObject swizzleInstanceMethodWithOriginSel:@selector(objectAtIndex:)
                                         swizzledSel:@selector(_wb_safe_ObjectAtIndex:)
                                           selfClass:NSClassFromString(@"__NSArrayI")];
        [NSObject swizzleInstanceMethodWithOriginSel:@selector(objectAtIndexedSubscript:)
                                         swizzledSel:@selector(_wb_safe_objectAtIndexedSubscript:)
                                           selfClass:NSClassFromString(@"__NSArrayI")];
    });
}


#pragma mark < Exchage Method >
- (id)_wb_safe_ZeroObjectAtIndex:(NSUInteger)index {
    if (index >= self.count) {
        return nil;
    }
    return [self _wb_safe_ZeroObjectAtIndex:index];
}

- (id)_wb_safe_ObjectAtIndex:(NSUInteger)index {
    if (index >= self.count) {
        return nil;
    }
    return [self _wb_safe_ObjectAtIndex:index];
}

- (id)_wb_safe_singleObjectAtIndex:(NSUInteger)index {
    if (index >= self.count) {
        return nil;
    }
    return [self _wb_safe_singleObjectAtIndex:index];
}

- (id)_wb_safe_objectAtIndexedSubscript:(NSUInteger)index {
    if (index >= self.count) {
        return nil;
    }
    return [self _wb_safe_objectAtIndexedSubscript:index];
}
```
- 计算时间差，类似朋友圈动态时间显示需要用到
```
+ (NSString *)compareCurrentTimeWithTimeString:(NSString *)timeString {
    if (!timeString) return nil;
    NSDateFormatter *formatter = [[WBDateFormatterPool shareInstance] wb_dateFormatterWithFormat:@"yyyy-MM-dd HH:mm:ss"];
    NSDate *nowDate = [NSDate date];
    NSDate *compareDate = [formatter dateFromString:timeString];
    /** < 时间差转换成秒 > */
    long delta = (long)[nowDate timeIntervalSinceDate:compareDate];
    if (delta <= 0 )return timeString;
    if(delta / (60 * 60 * 24 * 365) > 0) return [NSString stringWithFormat:@"%ld年前", delta / (60 * 60 * 24 * 365)];
    if (delta / (60 * 60 * 24 * 30) > 0) return [NSString stringWithFormat:@"%ld月前", delta / (60 * 60 * 24 * 30)];
    if (delta / (60 * 60 * 24 * 7) > 0) return [NSString stringWithFormat:@"%ld周前", delta / (60 * 60 * 24 * 7)];
    if (delta / (60 * 60 * 24) > 0) return [NSString stringWithFormat:@"%ld天前", delta / (60 * 60 * 24)];
    if (delta / (60 * 60) > 0) return [NSString stringWithFormat:@"%ld小时前", delta / (60 * 60)];
    if (delta / (60) > 0) return [NSString stringWithFormat:@"%ld分钟前", delta / (60)];
    return @"刚刚";
}
```

### 四、UIKit
- 图片压缩上传
```
#pragma mark --------  图片压缩  --------
+ (NSData *)wb_compressOriginalImage:(UIImage *)image
                 toMaxDataSizeKBytes:(CGFloat)size {
    NSData *data = UIImageJPEGRepresentation(image, 1.0);
    CGFloat dataKBytes = data.length / 1000.0;
    CGFloat maxQuality = 0.9f;
    CGFloat lastData = dataKBytes;
    while (dataKBytes > size && maxQuality > 0.01f) {
        maxQuality = maxQuality - 0.01f;
        data = UIImageJPEGRepresentation(image, maxQuality);
        dataKBytes = data.length / 1000.0;
        if (lastData == dataKBytes) {
            break;
        }else{
            lastData = dataKBytes;
        }
    }
    return data;
}

- (NSData *)wb_compressImageWithQuality:(CGFloat)quality {
    CGSize size = [self resizeImageWithBoundary:kBoundary];
    UIImage *reImage = [self resizeImageWithSize:size];
    NSData *imageData = UIImageJPEGRepresentation(reImage, quality);
    NSLog(@"压缩后图片大小 = %luKB",imageData.length / 1000);
    return imageData;
}

- (CGSize)resizeImageWithBoundary:(CGFloat)boundary {
    CGFloat width = self.size.width;
    CGFloat height = self.size.width;
    
    if (width < boundary || height < boundary) return CGSizeMake(width, height);
    
    CGFloat ratio = MAX(width, height) / MIN(width, height);
    if (ratio <= 2) {
        // Set the larger value to the boundary, the smaller the value of the compression
        CGFloat x = MAX(width, height) / boundary;
        if (width > height) {
            width = boundary;
            height = height / x;
        }else {
            height = boundary;
            width = width / x;
        }
    }else {
        // width, height > 1280
        if (MIN(width, height) >= boundary) {
            // Set the smaller value to the boundary, and the larger value is compressed
            CGFloat x = MIN(width, height) / boundary;
            if (width < height) {
                width = boundary;
                height = height / x;
            }else {
                height = boundary;
                width = width / x;
            }
        }
    }
    return CGSizeMake(width, height);
}

- (UIImage *)resizeImageWithSize:(CGSize)size {
    CGRect rect = CGRectMake(0, 0, size.width, size.height);
    UIImage *newImage;
    UIGraphicsBeginImageContext(rect.size);
    newImage = [UIImage imageWithCGImage:self.CGImage
                                   scale:1.f
                             orientation:self.imageOrientation];
    [newImage drawInRect:rect];
    newImage = UIGraphicsGetImageFromCurrentImageContext();
    UIGraphicsEndImageContext();
    return newImage;
}
```
- 扩大按钮点击区域
```
- (UIEdgeInsets)wb_touchAreaInsets
{
    return [objc_getAssociatedObject(self, @selector(wb_touchAreaInsets)) UIEdgeInsetsValue];
}

/**
 *  @brief  设置按钮额外热区
 */
- (void)setWb_touchAreaInsets:(UIEdgeInsets)touchAreaInsets
{
    NSValue *value = [NSValue valueWithUIEdgeInsets:touchAreaInsets];
    objc_setAssociatedObject(self, @selector(wb_touchAreaInsets), value, OBJC_ASSOCIATION_RETAIN_NONATOMIC);
}

- (BOOL)pointInside:(CGPoint)point withEvent:(UIEvent *)event
{
    UIEdgeInsets touchAreaInsets = self.wb_touchAreaInsets;
    CGRect bounds = self.bounds;
    bounds = CGRectMake(bounds.origin.x - touchAreaInsets.left,
                        bounds.origin.y - touchAreaInsets.top,
                        bounds.size.width + touchAreaInsets.left + touchAreaInsets.right,
                        bounds.size.height + touchAreaInsets.top + touchAreaInsets.bottom);
    return CGRectContainsPoint(bounds, point);
}
```
- 按钮倒计时，GCD定时器实现
```
- (void)wb_startTime:(NSInteger )timeout
               title:(NSString *)tittle
          waitTittle:(NSString *)waitTittle {
    __block NSInteger timeOut=timeout; //倒计时时间
    dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
    dispatch_source_t _timer = dispatch_source_create(DISPATCH_SOURCE_TYPE_TIMER, 0, 0,queue);
    dispatch_source_set_timer(_timer,dispatch_walltime(NULL, 0),1.0*NSEC_PER_SEC, 0); //每秒执行
    dispatch_source_set_event_handler(_timer, ^{
        if(timeOut<=0){ //倒计时结束，关闭
            dispatch_source_cancel(_timer);
            dispatch_async(dispatch_get_main_queue(), ^{
                //设置界面的按钮显示 根据自己需求设置
                [self setTitle:tittle forState:UIControlStateNormal];
                self.userInteractionEnabled = YES;
            });
        }else{
            //            int minutes = timeout / 60;
            int seconds = timeOut % 60;
            NSString *strTime = [NSString stringWithFormat:@"%.2d", seconds];
            dispatch_async(dispatch_get_main_queue(), ^{
                //设置界面的按钮显示 根据自己需求设置
                NSLog(@"____%@",strTime);
                [self setTitle:[NSString stringWithFormat:@"%@%@",strTime,waitTittle] forState:UIControlStateNormal];
                self.userInteractionEnabled = NO;
                
            });
            timeOut--;
            
        }
    });
    dispatch_resume(_timer);
}
```
- SDWebImageView设置图片交叉渐变效果
```
- (void)wb_setImageWithFadeAnimation:(NSString *)url
                    placeholderImage:(NSString *)placeholder
                            duration:(CGFloat)duration {
    __weak typeof(self) weakSelf = self;
    [self sd_setImageWithURL:[NSURL URLWithString:url]
            placeholderImage:[UIImage imageNamed:placeholder]
                   completed:^(UIImage * _Nullable image, NSError * _Nullable error, SDImageCacheType cacheType, NSURL * _Nullable imageURL) {
                       if (image && cacheType == SDImageCacheTypeNone) {
                           /**  < 添加交叉渐变动画 >  */
                           CATransition *animation = [CATransition animation];
                           animation.type = kCATransitionFade;
                           animation.duration = duration;
                           [weakSelf.layer addAnimation:animation forKey:@"fadeAnimation"];
                       }
    }];
}
```
- 获取设备uuid
```
/**
 also known as udid/uniqueDeviceIdentifier but this doesn't persists to system reset,we can use it to identifier user.

 @return uuid string.
 */
- (NSString *)wb_uuid;
```
- 根据颜色生成图片
```
+ (nullable UIImage *)wb_imageWithColor:(UIColor *)color
                                   size:(CGSize)size {
    if (!color || size.width <= 0 || size.height <= 0) return nil;
    CGRect rect = CGRectMake(0.0f, 0.0f, size.width, size.height);
    UIGraphicsBeginImageContextWithOptions(rect.size, NO, 0);
    CGContextRef context = UIGraphicsGetCurrentContext();
    CGContextSetFillColorWithColor(context, color.CGColor);
    CGContextFillRect(context, rect);
    UIImage *image = UIGraphicsGetImageFromCurrentImageContext();
    UIGraphicsEndImageContext();
    return image;
}
```
- 从十六进制字符串获取颜色
```
+ (UIColor *)wb_colorWithHexString:(NSString *)color
                             alpha:(CGFloat)alpha {
    //删除字符串中的空格
    NSString *cString = [[color stringByTrimmingCharactersInSet:[NSCharacterSet whitespaceAndNewlineCharacterSet]] uppercaseString];
    // String should be 6 or 8 characters
    if ([cString length] < 6)
    {
        return [UIColor clearColor];
    }
    // strip 0X if it appears
    //如果是0x开头的，那么截取字符串，字符串从索引为2的位置开始，一直到末尾
    if ([cString hasPrefix:@"0X"])
    {
        cString = [cString substringFromIndex:2];
    }
    //如果是#开头的，那么截取字符串，字符串从索引为1的位置开始，一直到末尾
    if ([cString hasPrefix:@"#"])
    {
        cString = [cString substringFromIndex:1];
    }
    if ([cString length] != 6)
    {
        return [UIColor clearColor];
    }
    
    // Separate into r, g, b substrings
    NSRange range;
    range.location = 0;
    range.length = 2;
    //r
    NSString *rString = [cString substringWithRange:range];
    //g
    range.location = 2;
    NSString *gString = [cString substringWithRange:range];
    //b
    range.location = 4;
    NSString *bString = [cString substringWithRange:range];
    
    // Scan values
    unsigned int r, g, b;
    [[NSScanner scannerWithString:rString] scanHexInt:&r];
    [[NSScanner scannerWithString:gString] scanHexInt:&g];
    [[NSScanner scannerWithString:bString] scanHexInt:&b];
    return [UIColor colorWithRed:((float)r / 255.0f) green:((float)g / 255.0f) blue:((float)b / 255.0f) alpha:alpha];
}
```
- 设置UITextView占位文字，通过Runtime实现
```
@property (nonatomic, strong) IBInspectable NSString *placeholder;
@property (nonatomic, strong) NSAttributedString *attributedPlaceholder;
@property (nonatomic, strong) IBInspectable UIColor *placeholderColor;

+ (UIColor *)defaultPlaceholderColor;
```
### 五、QuartzCore
- spring动画
```
- (void)wb_springAnimation:(NSTimeInterval)duration {
    CAKeyframeAnimation *popAnimation = [CAKeyframeAnimation animationWithKeyPath:@"transform"];
    popAnimation.duration = duration;
    popAnimation.values = @[[NSValue valueWithCATransform3D:CATransform3DMakeScale(0.01f, 0.01f, 1.0f)],
                            [NSValue valueWithCATransform3D:CATransform3DMakeScale(1.1f, 1.1f, 1.0f)],
                            [NSValue valueWithCATransform3D:CATransform3DMakeScale(0.9f, 0.9f, 1.0f)],
                            [NSValue valueWithCATransform3D:CATransform3DIdentity]];
    popAnimation.keyTimes = @[@0.2f, @0.5f, @0.75f, @1.0f];
    popAnimation.timingFunctions = @[[CAMediaTimingFunction functionWithName:kCAMediaTimingFunctionEaseInEaseOut],
                                     [CAMediaTimingFunction functionWithName:kCAMediaTimingFunctionEaseInEaseOut],
                                     [CAMediaTimingFunction functionWithName:kCAMediaTimingFunctionEaseInEaseOut]];
    popAnimation.removedOnCompletion = YES;
    [[self layer] addAnimation:popAnimation forKey:nil];
}
```
### 六、Marco
- 设置平方字体
```
/**  < 设置平方字体PingFangSC >  */
#define kWB_PFR kWB_SYSTEM_VERSION_9_OR_LATER ? @"PingFangSC-Regular" : @"PingFang SC"
```
- 清除警告宏
```
/** << 忽略PerformSelector警告 > */
#define SUPPRESS_PerformSelectorLeak_WARNING(Stuff) \
do { \
_Pragma("clang diagnostic push") \
_Pragma("clang diagnostic ignored \"-Warc-performSelector-leaks\"") \
Stuff; \
_Pragma("clang diagnostic pop") \
} while (0)

/** << 忽略未定义方法警告 > */
#define  SUPPRESS_Undeclaredselector_WARNING(Stuff) \
do { \
_Pragma("clang diagnostic push") \
_Pragma("clang diagnostic ignored \"-Wundeclared-selector\"") \
Stuff; \
_Pragma("clang diagnostic pop") \
} while (0)

/** << 忽略过期API警告 > */
#define SUPPRESS_DEPRECATED_WARNING(Stuff) \
do { \
_Pragma("clang diagnostic push") \
_Pragma("clang diagnostic ignored \"-Wdeprecated-declarations\"") \
Stuff; \
_Pragma("clang diagnostic pop") \
} while (0)
```
- iOS 11滚动视图适配
```
/**  < Adaptive  >  */
#define  kWB_AdjustsScrollViewInsets_NO(scrollView,vc)\
do { \
_Pragma("clang diagnostic push") \
_Pragma("clang diagnostic ignored \"-Warc-performSelector-leaks\"") \
if ([UIScrollView instancesRespondToSelector:NSSelectorFromString(@"setContentInsetAdjustmentBehavior:")]) {\
[scrollView   performSelector:NSSelectorFromString(@"setContentInsetAdjustmentBehavior:") withObject:@(2)];\
} else {\
vc.automaticallyAdjustsScrollViewInsets = NO;\
}\
_Pragma("clang diagnostic pop") \
} while (0)
```
- 适配宏
```
/**  < 屏幕适配 ipone6/6s 控件宽高 字体大小都可以用这个宏 >  */
#define kWB_AUTOLAYOUTSIZE(size) ((size) * (SCREEN_WIDTH / 375))
```
- iPhone X相关
```
/**  < 判断是否是iPhone X >  */
#define kWB_IS_IPHONE_X ([UIScreen instancesRespondToSelector:@selector(currentMode)] ? CGSizeEqualToSize(CGSizeMake(1125, 2436), [[UIScreen mainScreen] currentMode].size) : NO)

/**  < 导航栏高度 无largeTitle >  */
#define kWB_NAVIGATIONBAR_HEIGHT 44
/**  < 状态栏高度 >  */
#define kWB_STATUSBAR_HEIGHT [UIApplication sharedApplication].statusBarFrame.size.height
/**  < 整个导航栏高度 >  */
#define kWB_NAVI_HEIGHT (kWB_IS_IPHONE_X ? (88) : (64))
/**  < 标签栏高度 >  */
#define kWB_TABBAR_HEIGHT (kWB_IS_IPHONE_X ? (83) : (49))
/**  < iOS 11 底部安全域距离 >  */
#define kWB_BOTTOM_SAFEAREA_HEIGHT (kWB_IS_IPHONE_X ? (34) : (0))

/**  < 判断 iOS 11 或更高的系统版本 >  */
#define kWB_SYSTEM_VERSION_11_OR_LATER SYSTEM_VERSION_GREATER_THAN_OR_EQUAL_TO(@"11.0")

```
- 单例宏定义
```
/**  < 单例ARC&MRC宏定义 >  */
/**  < .h >  */
#define singletonH(name) +(instancetype)share##name;

/**  < .m >  */
#if __has_feature(objc_arc)
//ARC
#define singleM(name) static id _instance;\
+(instancetype)allocWithZone:(struct _NSZone *)zone\
{\
static dispatch_once_t onceToken;\
dispatch_once(&onceToken, ^{\
_instance = [super allocWithZone:zone];\
});\
return _instance;\
}\
\
+(instancetype)share##name\
{\
return [[self alloc]init];\
}\
-(id)copyWithZone:(NSZone *)zone\
{\
return _instance;\
}\
\
-(id)mutableCopyWithZone:(NSZone *)zone\
{\
return _instance;\
}

#else
//非ARC
#define singleM static id _instance;\
+(instancetype)allocWithZone:(struct _NSZone *)zone\
{\
static dispatch_once_t onceToken;\
dispatch_once(&onceToken, ^{\
_instance = [super allocWithZone:zone];\
});\
return _instance;\
}\
\
+(instancetype)shareTools\
{\
return [[self alloc]init];\
}\
-(id)copyWithZone:(NSZone *)zone\
{\
return _instance;\
}\
-(id)mutableCopyWithZone:(NSZone *)zone\
{\
return _instance;\
}\
-(oneway void)release\
{\
}\
\
-(instancetype)retain\
{\
return _instance;\
}\
\
-(NSUInteger)retainCount\
{\
return MAXFLOAT;\
}
#endif
```
- Block防止循环引用宏定义
```
#ifndef weakify
    #if DEBUG
        #if __has_feature(objc_arc)
        #define weakify(object) autoreleasepool{} __weak __typeof__(object) weak##_##object = object;
        #else
        #define weakify(object) autoreleasepool{} __block __typeof__(object) block##_##object = object;
        #endif
    #else
        #if __has_feature(objc_arc)
        #define weakify(object) try{} @finally{} {} __weak __typeof__(object) weak##_##object = object;
        #else
        #define weakify(object) try{} @finally{} {} __block __typeof__(object) block##_##object = object;
        #endif
    #endif
#endif

#ifndef strongify
    #if DEBUG
        #if __has_feature(objc_arc)
        #define strongify(object) autoreleasepool{} __typeof__(object) object = weak##_##object;
        #else
        #define strongify(object) autoreleasepool{} __typeof__(object) object = block##_##object;
        #endif
    #else
        #if __has_feature(objc_arc)
        #define strongify(object) try{} @finally{} __typeof__(object) object = weak##_##object;
        #else
        #define strongify(object) try{} @finally{} __typeof__(object) object = block##_##object;
        #endif
    #endif
#endif
```
### 七、结语
上面这些方法是我在项目中使用的较多的方法，当然分类方法也不止我贴出的这些，有兴趣的朋友可以到我的[GitHub](https://github.com/wenmobo)上查看，[WBCategories](https://github.com/wenmobo/WBCategories)有些分类是我自己写的，如果有写的不对的地方，欢迎批评指正，我也会第一时间修改，如果对你有帮助也请Star一个吧。Finally，I will continue update it!
### 参考博客
- [iOS 关于Category](https://www.jianshu.com/p/535d1574cb86)
- [ios-category解析](https://www.jianshu.com/p/e917e7d95f69)