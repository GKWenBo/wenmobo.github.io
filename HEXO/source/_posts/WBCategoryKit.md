---
title: WBCategoryKit
tags: [iOS,Objective-C,CocoaPods,GitHub]
date: 2018-07-08 17:26:13
permalink:
categories: iOS
description: 在日常开发过程中，我们也经常用到分类。分类（Category）是OC中的特有语法，它是表示一个指向分类的结构体的指针。原则上它只能增加方法，不能增加成员（实例）变量
image:
---
<p class="description"></p>

<!--more-->

## 中文说明
> Some useful Objective-C ategories and Macro,that contain UIKit.framework、Foundation.framework、AVFoundation.framework、QuartzCore. framework、CoreTelephony.framework、WebKit.framework、MobileCoreServices.framework、Photos.framework、AssetsLibrary.framework、Accelerate.framework、ImageIO.framework、CoreText.framework、CoreGraphics.framework and so on,i will continue to tidy up updates.

 iOS 系统常用框架分类封装，开发常用宏定义，支持cocoapod集成，支持只集成子模块。持续更新中...

## Requirements

- iOS 8+
- Xcode 8+

## Installation

### Cocoapods安装
- 安装所有分类文件
```ruby
pod 'WBCategoryKit'
```
- 集成子组件
```ruby
pod 'WBCategoryKit/UIKit'
```
或者
```ruby
pod 'WBCategoryKit/UIKit/WKWebView'
```

### 手动集成

将需要的分类文件拖入工程即可。

## Usage

### Foundation

- NSObject     
```
//swizzle 类方法
+ (void)swizzleClassMethodWithOriginSel:(SEL)oriSel
                            swizzledSel:(SEL)swiSel
                              selfClass:(Class)selfClass;
```

```
//swizzle 实例方法 
+ (void)swizzleInstanceMethodWithOriginSel:(SEL)oriSel
                               swizzledSel:(SEL)swiSel
                                 selfClass:(Class)selfClass;
```

- NSDate    
```
//NSDateFormatter缓存 
- (NSDateFormatter *)wb_dateFormatterWithFormat:(NSString *)format;
```

//朋友圈时间格式   
```
+ (NSString *)compareCurrentTimeWithTimeString:(NSString *)timeString;
```

### Macro

//设置平方字体PingFangSC  
```
#define kWB_PFR kWB_SYSTEM_VERSION_9_OR_LATER ? @"PingFangSC-Regular" : @"PingFang SC"
#define kWB_PFR_FONT(s) [UIFont fontWithName:kWB_PFR size:s]
```

//主线程安全执行   
```
#ifndef dispatch_main_async_safe
#define dispatch_main_async_safe(block) dispatch_queue_async_safe(dispatch_get_main_queue(), block)
#endif
```

//同步锁   
```
#define kWB_LOCK(lock) dispatch_semaphore_wait(lock, DISPATCH_TIME_FOREVER);
#define kWB_UNLOCK(lock) dispatch_semaphore_signal(lock);
```

### UIKit

- WKWebView    
```
//获取某个标签的结点个数
- (void)wb_nodeCountOfTag:(NSString *)tag
completedHandler:(void (^) (int tagCount))completedHandler;
```

```
//获取网页中的图片 
- (void)wb_getImages:(void (^) (NSArray *images))completedHandler;
```

```
//获取网页内容高度
- (void)wb_getScrollHeight:(void (^) (CGFloat scrollHeight))completedHandler;
```

```
//为所有图片添加点击事件
- (void)wb_addClickEventOnImg;
```

```
//根据id隐藏网页元素
- (void)wb_hiddenElementById:(NSString *)idString;
```

- UIFont       
```
//runtime字体适配
+ (UIFont *)_wb_systemFontOfSize:(CGFloat)fontSize;
+ (UIFont *)_wb_boldSystemFontOfSize:(CGFloat)fontSize;
+ (UIFont *)_wb_fontWithName:(NSString *)fontName
                        size:(CGFloat)fontSize;
```
 更多分类使用方法，请查看[WBCategoryKit](https://github.com/wenmobo/WBCategoryKit)。
## 补充
本库主要是记录自己积累学习的一个过程，最初在github创建这个工程的时候，我就在自己的博客中写道将来有一天将本库制作成pod公有库，如今完成了本库的制作，虽然在制作过程中遇到了很多的问题，但还是很有成就感。如过在使用过程中，有任何建议或者问题，可以通过以下方式联系我，十分感谢。

author：wenbo    
     QQ：1050794513  
  email：1050794513@qq.com    
 喜欢就❤️下鼓励下吧。
## 更新 
> - 2018-09-05 （1.0.2）： 修改podspec文件，支持三级目录。

- https://www.jianshu.com/p/e917e7d95f69)