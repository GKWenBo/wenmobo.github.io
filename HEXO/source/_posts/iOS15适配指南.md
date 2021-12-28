---
title: iOS15适配指南
tags: [iOS15,Xcode]
date: 2021-09-02 22:51:29
permalink:
categories: iOS
description: iOS15适配指南
image:
---
<p class="description"></p>

<img src="https://" alt="" style="width:100%" />

<!-- more -->

### UITableView Group样式多出间距处理

项目部分界面会有这个问题，解决方案如下

```swift
tableView.tableHeaderView = UIView.init(frame: CGRect(x: 0, y: 0, width: UIScreen.main.bounds.size.width, height: CGFloat(Float.leastNormalMagnitude)))
```

### UITableView新增API导致顶部多出间距

```swift
/// Padding above each section header. The default value is `UITableViewAutomaticDimension`.
    @available(iOS 15.0, *)
    open var sectionHeaderTopPadding: CGFloat
```

解决方法，直接设置成0即可

```swift
tableView.sectionHeaderTopPadding = 0;
```

### 原生导航栏透明

在真机iOS beta7系统上，项目导航栏默认透明

```objective-c
bar.barTintColor = self.navigationController.navigationBar.barTintColor;
    [bar setBackgroundImage:[self.navigationController.navigationBar backgroundImageForBarMetrics:UIBarMetricsDefault] forBarMetrics:UIBarMetricsDefault];
    bar.shadowImage = self.navigationController.navigationBar.shadowImage;
```

需用到iOS13相关API进行适配

```objective-c
#ifdef __IPHONE_15_0
    if (@available(iOS 15.0, *)) {
        self.km_transitionBarAppearance.backgroundColor = bar.barTintColor;
        UIImage *backgroundImage = [bar backgroundImageForBarMetrics:UIBarMetricsDefault];
        self.km_transitionBarAppearance.backgroundImage = backgroundImage;
    
        UIImage *shadowImage = bar.shadowImage;
        if (shadowImage && shadowImage.size.width <= 0 && shadowImage.size.height <= 0) {
            shadowImage = nil;
            self.km_transitionBarAppearance.shadowColor = [UIColor clearColor];
        }
        self.km_transitionBarAppearance.shadowImage = shadowImage;
        if (bar.titleTextAttributes) {
            self.km_transitionBarAppearance.titleTextAttributes = bar.titleTextAttributes;
        }
        
        self.navigationController.navigationBar.scrollEdgeAppearance = self.km_transitionBarAppearance;
        self.navigationController.navigationBar.standardAppearance = self.km_transitionBarAppearance;
    }
#endif
```

### KMNavigationBarTransition导航栏转场动画库样式问题

在真机iOS beta7系统上，页面跳转转场动画会有异常现象，也需进行适配，我对仓库进行了fork，修改了部分代码，仅供参考：

https://github.com/GKWenBo/KMNavigationBarTransition

同理，如果用到[QMUI_iOS](https://github.com/Tencent/QMUI_iOS)中的转场库，也需进行适配处理。

### 高德地图存在崩溃问题

官方已经对SDK进行修复发布新版。

> 影响APP体验和稳定性的暂时就发现这些，后面有遇到新的问题也会进行补充，iOS15一些新API就自己去了解学习吧。

<hr />
