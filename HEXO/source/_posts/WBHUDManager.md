---
title: WBHUDManager
tags: [MBProgressHUD,iOS,GitHub,Objective-C,Swift]
date: 2018-07-08 18:26:42
permalink:
categories: iOS
description:
image: https://ws3.sinaimg.cn/large/006tNc79ly1ft2mqzgb4zj31ii0pe0zb.jpg
---
<p class="description"></p>

<!-- more -->

### 更新日志

2018-08-01：更新API，支持配置更多自定义设置，录制GIF。

2018-09-09：支持pod安装

### 前言

> 在我们平时做项目的时候，为了提高交互体验，难免会用到一些提示语。除了UI上有特殊的要求需要自定义提示UI，一般会选择[GitHub](https://github.com/)上一些知名的提示框架库，如：
- [MBProgressHUD](https://github.com/jdg/MBProgressHUD)
- [SVProgressHUD](https://github.com/SVProgressHUD/SVProgressHUD)
- [JGProgressHUD](https://github.com/JonasGessner/JGProgressHUD)
- [Toast](https://github.com/scalessec/Toast)

之前做项目都是用的**SVProgressHUD**，这个三方提示库使用非常接单，基于这个库，也很好做自定义提示封装。后来做项目改成了MBProgressHUD，相对于SVProgressHUD，MBProgressHUD使用相对来说要麻烦一点，因此，我对MBProgressHUD一些常用提示进行了封装，最开始封装的工具类存在着一些缺点，比如说在网络请求的时候，如果网络不好，拿不到回调，MBProgressHUD就会一直显示，用户无法交互，因此对这个工具类进行了改进。
### MBProgressHUD(v1.1.0)
我主要写了一个分类，有菊花、文字、文字+图片提示，并提供了显示完成对调，方便显示完成后进行相应的操作或界面跳转，提供的调用方法如下：
```objective-c
// MARK:Loading
/**
 只显示菊花，不会自动消失 (白字+黑底)

 @return MBProgressHUD实例对象
 */
+ (MBProgressHUD *)wb_showActivity;

/**
 只显示菊花，不会自动消失 (白字+黑底+自定义视图)

 @param view 要显示的视图
 @return MBProgressHUD实例对象
 */
+ (MBProgressHUD *)wb_showActivity:(UIView *)view;

/**
 菊花+文字 (白字+黑底)

 @param message 加载文字
 @return  MBProgressHUD实例对象
 */
+ (MBProgressHUD *)wb_showActivityMessage:(NSString *)message;

/**
 菊花+文字 (白字+黑底)

 @param message 加载文字
 @param view 要显示的视图
 @return MBProgressHUD实例对象
 */
+ (MBProgressHUD *)wb_showActivityMessage:(NSString *)message
                                   toView:(UIView *)view;

/**
 菊花+文字 （自定义文字+内容颜色+蒙版颜色+容器颜色）

 @param message 加载文字
 @param view 要显示的视图
 @param contentColor 内容颜色
 @param maskColor 蒙版颜色
 @param bezelColor 容器颜色
 @return MBProgressHUD实例对象
 */
+ (MBProgressHUD *)wb_showActivityMessage:(NSString *)message
                                   toView:(UIView *)view
                             contentColor:(UIColor *)contentColor
                                maskColor:(UIColor *)maskColor
                               bezelColor:(UIColor *)bezelColor;

/**
 菊花+文字 （自定义文字+文字颜色+蒙版颜色+容器颜色）

 @param message 加载文字
 @param view 要显示的视图
 @param titleColor 文字颜色
 @param maskColor 蒙版颜色
 @param bezelColor 容器颜色
 @return MBProgressHUD实例对象
 */
+ (MBProgressHUD *)wb_showActivityMessage:(NSString *)message
                                   toView:(UIView *)view
                               titleColor:(UIColor *)titleColor
                                maskColor:(UIColor *)maskColor
                               bezelColor:(UIColor *)bezelColor;

// MARK:Text
/**
 提示文字 （自定义文+位置中间 + 显示在window）

 @param message 文字
 */
+ (void)wb_showMessage:(NSString *)message;

/**
 提示文字 (标题 + 详情文字)

 @param message 文字
 @param detailMessage 详情文字
 */
+ (void)wb_showMessage:(NSString *)message
         detailMessage:(NSString *)detailMessage;

/**
 提示文字 (标题 + 详情文字 + 自定义位置 + 视图)

 @param message 文字
 @param detailMessage 详情文字
 @param position 位置
 */
+ (void)wb_showMessage:(NSString *)message
         detailMessage:(NSString *)detailMessage
                toView:(UIView *)view
              position:(WBHUDPositionStyle)position;

/**
 提示文字（自定义文+位置中间+显示在window+完成回调）

 @param message 文字
 @param completion 完成回调
 */
+ (void)wb_showMessage:(NSString *)message
            completion:(MBProgressHUDCompletionBlock)completion;

/**
 提示文字（自定文字+位置中间+自定义显示视图+完成回调）

 @param message 文字
 @param view 要显示的视图
 @param completion 完成回调
 */
+ (void)wb_showMessage:(NSString *)message
                toView:(UIView *)view
            completion:(MBProgressHUDCompletionBlock)completion;

/**
 提示文字 (自定义文字+自定义位置+自定义显示视图)

 @param message 文字
 @param view 要显示的视图
 @param position 位置
 @param completion 完成回调
 */
+ (void)wb_showMessage:(NSString *)message
                toView:(UIView *)view
              position:(WBHUDPositionStyle)position
            completion:(MBProgressHUDCompletionBlock)completion;

/**
 提示文字 (自定义文字+详情文字+自定义位置+内容样式)

 @param message 文字
 @param detailTitle 详情文字
 @param view 要显示的视图
 @param position 显示位置
 @param contentStyle 内容样式
 @param completion 完成回调
 */
+ (void)wb_showMessage:(NSString *)message
           detailTitle:(NSString *)detailTitle
                toView:(UIView *)view
              position:(WBHUDPositionStyle)position
          contentStyle:(WBHUDContentStyle)contentStyle
            completion:(MBProgressHUDCompletionBlock)completion;

// MARK:Image

/**
 自定义成功提示 (显示在window)

 @param success 提示文字
 */
+ (void)wb_showSuccess:(NSString *)success;

/**
 自定义成功提示 (显示在window + 完成回调)

 @param success 提示文字
 @param completion 完成回调
 */
+ (void)wb_showSuccess:(NSString *)success
            completion:(MBProgressHUDCompletionBlock)completion;

/**
 自定义成功提示 (显示在window + 完成回调 + 自定义显示视图)

 @param success 提示文字
 @param view 显示视图
 @param completion 完成回调
 */
+ (void)wb_showSuccess:(NSString *)success
                toView:(UIView *)view
            completion:(MBProgressHUDCompletionBlock)completion;

/**
 错误提示 (显示在window)

 @param error 提示文字
 */
+ (void)wb_showError:(NSString *)error;

/**
 错误提示 (显示在window + 完成回调)

 @param error 错误提示
 @param completion 完成回调
 */
+ (void)wb_showError:(NSString *)error
          completion:(MBProgressHUDCompletionBlock)completion;

/**
 错误提示 (显示在window + 完成回调 + 自定义显示视图)

 @param error 错误提示
 @param view 示视图
 @param completion 完成回调
 */
+ (void)wb_showError:(NSString *)error
              toView:(UIView *)view
          completion:(MBProgressHUDCompletionBlock)completion;

/**
 信息提示 (window)

 @param info 提示文字
 */
+ (void)wb_showInfo:(NSString *)info;

/**
 信息提示 (window + 完成回调)

 @param info 提示文字
 @param completion 完成回调
 */
+ (void)wb_showInfo:(NSString *)info
         completion:(MBProgressHUDCompletionBlock)completion;

/**
 信息提示 (window + 完成回调 + 自定义显示视图)

 @param info 提示文字
 @param view 自定义显示视图
 @param completion 完成回调
 */
+ (void)wb_showInfo:(NSString *)info
             toView:(UIView *)view
         completion:(MBProgressHUDCompletionBlock)completion;

/**
 警告提示 (window)

 @param warning 提示文字
 */
+ (void)wb_showWarning:(NSString *)warning;

/**
  警告提示 (window + 完成回调)

 @param warning 警告
 @param completion 完成回调
 */
+ (void)wb_showWarning:(NSString *)warning
            completion:(MBProgressHUDCompletionBlock)completion;

/**
 警告提示 (window + 完成回调 + 自定义视图)

 @param warning 警告
 @param view 自定义视图
 @param completion 完成回调
 */
+ (void)wb_showWarning:(NSString *)warning
                toView:(UIView *)view
            completion:(MBProgressHUDCompletionBlock)completion;

/**
 自定义图片 + 文字提示

 @param text 文字
 @param icon 图片名
 @param view 要显示的视图
 @param completion 完成回调
 */
+ (void)wb_show:(NSString *)text
           icon:(NSString *)icon
           view:(UIView *)view
     completion:(MBProgressHUDCompletionBlock)completion;

// MARK:Switch Model
/**
 Model切换

 @param view 要显示的视图
 @param title 要显示的文字
 @param configBlock 配置hud
 @return MBProgressHUD实例对象
 */
+ (MBProgressHUD *)wb_showModelSwitch:(UIView *)view
                                title:(NSString *)title
                          configBlock:(WBHUDConfigBlock)configBlock;

// MARK:Progress
/**
 文字 + 进度条

 @param view 要显示的视图
 @param progressStyle 进度样式
 @param title 提示文字
 @param configBlock 进度配置block
 @return MBProgressHUD实例对象
 */
+ (MBProgressHUD *)wb_showDownloadToView:(UIView *)view
                           progressStyle:(WBHUDProgressStyle)progressStyle
                                   title:(NSString *)title
                             configBlock:(WBHUDConfigBlock)configBlock;

/**
 文字 + 进度条 + 取消按钮

 @param view 要显示的视图
 @param progressStyle 进度样式
 @param title 提示文字
 @param cancelTitle 取消按钮标题
 @param configBlock 进度配置block
 @param cancelBlock 取消按钮点击回调
 @return MBProgressHUD实例对象
 */
+ (MBProgressHUD *)wb_showDownloadToView:(UIView *)view
                           progressStyle:(WBHUDProgressStyle)progressStyle
                                   title:(NSString *)title
                             cancelTitle:(NSString *)cancelTitle
                             configBlock:(WBHUDConfigBlock)configBlock
                             cancelBlock:(WBHUDCancelBlock)cancelBlock;

// MARK:Hide
+ (void)wb_hideHUD;
+ (void)wb_hideHUDForView:(UIView *)view;
```
举一个.m显示菊花方法的例子吧，MBProgressHUD最新版本对比老版本API还是有些变化的：
```
/** < 创建HUD > */
+ (MBProgressHUD *)wb_createHUD:(UIView *)view {
    if (view == nil) view = (UIView *)[UIApplication sharedApplication].delegate.window;
    return [MBProgressHUD showHUDAddedTo:view
                                animated:YES];
}

/** < 设置HUD > */
+ (MBProgressHUD *)wb_configHUDWithView:(UIView *)view
                                  title:(NSString *)title
                            autoDismiss:(BOOL)autoDismiss
                             completion:(MBProgressHUDCompletionBlock)completion {
    MBProgressHUD *hud = [self wb_createHUD:view];
    /** < 自动换行 > */
    hud.label.numberOfLines = 0;
    /** < 提示文字 > */
    hud.title(title);
    /** < 隐藏移除 > */
    hud.removeFromSuperViewOnHide = YES;
    /** <默认内容样式：黑底白字 > */
    hud.hudContentStyle(WBHUDContentBlackStyle);
    /** < 自动隐藏 > */
    if (autoDismiss) {
        [hud hideAnimated:YES
               afterDelay:KHideAfterDelayTime];
    }
    hud.completionBlock = completion;
    return hud;
}
```
感兴趣的朋友，可以下载Demo查看具体方法实现，请戳：[WBHUDManager](https://github.com/wenmobo/WBHUDManager)
![](https://ws4.sinaimg.cn/large/006tKfTcly1ftuhtkjtapg308p0i80x9.gif)
### 结语
> 选择哪一款提示框架，都要看自己喜好了，因为我代码水平有限，有些地方可能考虑的不够完善，只能说是抛砖引玉吧，大神们可能有更好的封装。要想基于这些框架自定义出自己需要风格的UI，还是要对框架提供的方法属性有一定的了解。