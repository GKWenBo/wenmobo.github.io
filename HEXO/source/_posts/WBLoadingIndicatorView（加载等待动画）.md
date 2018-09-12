---
title: WBLoadingIndicatorView（加载等待动画）
tags: [Animation,iOS,CocoaPods,GitHub]
date: 2018-09-11 22:27:00
permalink:
categories: iOS
description: 关于加载提示框架，比较成熟的有[MBProgressHUD](https://github.com/jdg/MBProgressHUD)，[SVProgressHUD](https://github.com/SVProgressHUD/SVProgressHUD)等著名框架，如果没有太多的自定义要求，这些框架提供的API其实已经够用了，基于提供的API，我们也可以自定义一些UI效果。最近项目也不是很忙，于是自己就尝试封装一个加载等待动画组件，封装思想主要参考了MBProgressHUD，在实现过程中，布局采用的是苹果原生Autolayout，没有用Masonry，所以写起来比较的恶心，约束写的老长老长了。其实封装的这个组件功能也不算太多，现在主要实现了五六个加载动画效果，也提供了一些属性设置自定义效果，如果有时间我会优化添加更多动画效果。
image:
---
<p class="description"></p>

<!-- more -->

### 一、前言
> 关于加载提示框架，比较成熟的有[MBProgressHUD](https://github.com/jdg/MBProgressHUD)，[SVProgressHUD](https://github.com/SVProgressHUD/SVProgressHUD)等著名框架，如果没有太多的自定义要求，这些框架提供的API其实已经够用了，基于提供的API，我们也可以自定义一些UI效果。最近项目也不是很忙，于是自己就尝试封装一个加载等待动画组件，封装思想主要参考了MBProgressHUD，在实现过程中，布局采用的是苹果原生Autolayout，没有用Masonry，所以写起来比较的恶心，约束写的老长老长了。其实封装的这个组件功能也不算太多，现在主要实现了五六个加载动画效果，也提供了一些属性设置自定义效果，如果有时间我会优化添加更多动画效果。

### 二、介绍与使用
- 一些属性API。
```
// MARK:Property
/*  < 动画容器视图 > */
@property (nonatomic, strong) WBLoadingBackgroundView *bezelView;
/*  < 背景视图 > */
@property (nonatomic, strong) WBLoadingBackgroundView *backgroundView;
/** < Loading text. > */
@property (nonatomic, strong) UILabel *label;

@property (nonatomic, strong) UIColor *contentColor UI_APPEARANCE_SELECTOR;
/*  < 加载动画颜色 > */
@property (nonatomic, strong) UIColor *indicatorColor UI_APPEARANCE_SELECTOR;
/*  < bezelView中心点偏移 > */
@property (nonatomic, assign) CGPoint offset UI_APPEARANCE_SELECTOR;
/*  < 边距 默认：20 > */
@property (nonatomic, assign) CGFloat margin UI_APPEARANCE_SELECTOR;
/*  < bezelView最小size > */
@property (nonatomic, assign) CGSize minSize UI_APPEARANCE_SELECTOR;
/** < 加载动画size 默认：35 > */
@property (nonatomic, assign) CGSize indicatorSize UI_APPEARANCE_SELECTOR;
/** < 是否方形 > */
@property (nonatomic, assign) BOOL square UI_APPEARANCE_SELECTOR;
/** < 隐藏时从父视图移除 默认：YES > */
@property (nonatomic, assign) BOOL removeFromSuperViewOnHide;
/*  < 动画类型 > */
@property (nonatomic, assign) WBLoadingAnimationType type;

// MARK:Class Methods
/**
 获取视图中的WBLoadingIndicatorView

 @param view 遍历的父视图
 @return WBLoadingIndicatorView
 */
+ (nullable WBLoadingIndicatorView *)wb_indicatorForView:(UIView *)view;

/**
 创建并显示加载视图

 @param view 要显示的view
 @return MBProgressHUD
 */
+ (instancetype)wb_showIndicatorAddTo:(UIView *)view;

// MARK:Instance Class Method
- (instancetype)initWithView:(UIView *)view;

/**
 显示加载视图
 */
- (void)wb_showLoadingView:(BOOL)animated;

/**
 隐藏加载视图

 @param animated 是否动画
 */
- (void)wb_hideLoadingView:(BOOL)animated;
```
- 部分效果
![](https://ws2.sinaimg.cn/large/0069RVTdly1fv5vfzum5xg308p0i2aak.gif)
![](https://ws4.sinaimg.cn/large/0069RVTdly1fv5vhwysnfg308p0i2dgd.gif)
![](https://ws4.sinaimg.cn/large/0069RVTdly1fv5vw05y8cg308p0i2wf5.gif)
![](https://ws1.sinaimg.cn/large/0069RVTdly1fv5vwliqtyg308p0i276d.gif)
- 使用示例
```
WBLoadingIndicatorView *indicatorView = [self createIndicatorViewWithType:WBWBLoadingAnimationBallTrianglePathType
                                                                        indicatorSize:CGSizeMake(50, 50)
                                                                               toView:self.view];
            indicatorView.type = WBLoadingAnimationcircleStrokeSpinType;
            indicatorView.backgroundView.backgroundColor = [UIColor colorWithWhite:0.f alpha:0.3];
            indicatorView.contentColor = [UIColor whiteColor];
            indicatorView.bezelView.backgroundColor = [UIColor colorWithWhite:0.f alpha:0.7f];
```
关于使用，建议还是二次封装吧，每次调用都写这么多代码，还是有点长。关于更多还是查看我的GitHub，下面也贴出GitHub地址。
### 三、GitHub地址
如果觉得可以，请star鼓励一下哦，如果有任何建议或问题，欢迎指出，我也会第一时间修改。
[WBLoadingIndicatorView](https://github.com/wenmobo/WBLoadingIndicatorView)






