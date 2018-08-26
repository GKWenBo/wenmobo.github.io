---
title: iOS状态栏配置
tags: [iOS,Runtime,Category]
date: 2018-07-26 22:57:56
permalink:
categories: iOS
description: 在APP开发当中，都要与状态栏打交道，因此我们需要了解采用哪种方式去管理状态栏样式，更容易开发与维护，如果界面定制地方较多，就不太适合全局配置状态栏，因为这样每次在界面将要显示、消失去配置状态栏，比较的繁琐，因此我们可以采用单个配置，不影响全局，而且可以随意切换样式。这篇文章主要也是记录自己在项目中遇到的问题，方便自己以后查阅，如果有写的不对的地方，希望能给予批评指正，我会及时更改。好了，开始介绍吧！
image: https://ws3.sinaimg.cn/large/006tKfTcly1ftnobbf8sxj30ho048jrh.jpg
---
<p class="description"></p>

<!-- more -->

### 一、全局配置状态栏

- 在**info.plist**中添加key：**View controller-based status bar appearance**，并设置value为**NO**。

- 在需要设置样式的地方调用：

  ```objective-c
  //UIStatusBarStyleLightContent：Light content, for use on dark backgrounds
  //UIStatusBarStyleDefault：Dark content, for use on light backgrounds
  [UIApplication sharedApplication].statusBarStyle = UIStatusBarStyleLightContent;
  ```

  

### 二、配置单个控制器状态栏

- 在**info.plist**中添加key：**View controller-based status bar appearance**，并设置value为**YES**。

- 如果有控制器没有导航控制器，直接重写**preferredStatusBarStyle** 方法返回你想要的状态栏样式即可：

  ```objective-c
  - (UIStatusBarStyle)preferredStatusBarStyle {
      return UIStatusBarStyleLightContent;
  }
  ```

- 通常都有导航控制器，如果控制器中直接重写**preferredStatusBarStyle**是没有效果的，这时需要在基类导航控制器中重写**childViewControllerForStatusBarStyle** 、**preferredStatusBarStyle** 任意一个方法就能实现配置单个控制器的状态栏样式：

  ```objective-c
  - (UIViewController *)childViewControllerForStatusBarStyle {
      return self.topViewController;
  }
  
  或者重写
  - (UIStatusBarStyle)preferredStatusBarStyle {
      return [self.topViewController preferredStatusBarStyle];
  }
  ```

  **注意**：两个方法都重写，只会调用**childViewControllerForStatusBarStyle**，所以需要自己根据清空去选择调用那个方法。

### 三、通过分类实现

- 在项目中方便使用，我写了一个**UINavigationController（WBStatusBarStyle）** 分类，主要代码如下：

  ```objective-c
  @implementation UINavigationController (WBStatusBarStyle)
  
  + (void)wb_setDefaultStatusBarStyle:(UIStatusBarStyle)statusBarStyle {
      objc_setAssociatedObject(self, &kWBDefaultStatusBarStyleKey, @(statusBarStyle), OBJC_ASSOCIATION_RETAIN_NONATOMIC);
  }
  
  + (UIStatusBarStyle)wb_DefaultStatusBarStyle {
      id style = objc_getAssociatedObject(self, &kWBDefaultStatusBarStyleKey);
      return style ? [style integerValue] : UIStatusBarStyleDefault;
  }
  
  /** < Override to return a child view controller or nil. If non-nil, that view controller's status bar appearance attributes will be used. If nil, self is used. Whenever the return values from these methods change, -setNeedsUpdatedStatusBarAttributes should be called. > */
  //- (UIViewController *)childViewControllerForStatusBarStyle {
  //    return self.topViewController;
  //}
  //
  //- (UIViewController *)childViewControllerForStatusBarHidden {
  //    return self.topViewController;
  //}
  
  - (UIStatusBarStyle)preferredStatusBarStyle {
      return [self.topViewController wb_statusBarStyle];
  }
  
  @end
  ```

- **UIViewController (WBStatusBarStyle)** 

```objective-c
@implementation UIViewController (WBStatusBarStyle)

- (void)setWb_statusBarStyle:(UIStatusBarStyle)wb_statusBarStyle {
    objc_setAssociatedObject(self, &kWBStatusBarStyleKey, @(wb_statusBarStyle), OBJC_ASSOCIATION_RETAIN_NONATOMIC);
    /** < Whenever the return values from these methods change, -setNeedsUpdatedStatusBarAttributes should be called. > */
    [self setNeedsStatusBarAppearanceUpdate];
}

- (UIStatusBarStyle)wb_statusBarStyle {
    id style = objc_getAssociatedObject(self, &kWBStatusBarStyleKey);
    return style ? [style integerValue] : [UINavigationController wb_DefaultStatusBarStyle];
}

@end
```

### 四、GitHub Demo

- [WBManageStatusBarStyleDemo](https://github.com/wenmobo/WBManageStatusBarStyleDemo)
- [WBCategories](https://github.com/wenmobo/WBCategories)