---
title: 最新MBProgressHUD封装
tags: [MBProgressHUD,iOS,GitHub,Objective-C,Swift]
date: 2018-07-08 18:26:42
permalink:
categories: iOS
description:
image: https://ws3.sinaimg.cn/large/006tNc79ly1ft2mqzgb4zj31ii0pe0zb.jpg
---
<p class="description"></p>

<!-- more -->

### 前言
> 在我们平时做项目的时候，为了提高交互体验，难免会用到一些提示语。除了UI上有特殊的要求需要自定义提示UI，一般会选择[GitHub](https://github.com/)上一些知名的提示框架库，如：
- [MBProgressHUD](https://github.com/jdg/MBProgressHUD)
- [SVProgressHUD](https://github.com/SVProgressHUD/SVProgressHUD)
- [JGProgressHUD](https://github.com/JonasGessner/JGProgressHUD)
- [Toast](https://github.com/scalessec/Toast)

之前做项目都是用的**SVProgressHUD**，这个三方提示库使用非常接单，基于这个库，也很好做自定义提示封装。后来做项目改成了MBProgressHUD，相对于SVProgressHUD，MBProgressHUD使用相对来说要麻烦一点，因此，我对MBProgressHUD一些常用提示进行了封装，最开始封装的工具类存在着一些缺点，比如说在网络请求的时候，如果网络不好，拿不到回调，MBProgressHUD就会一直显示，用户无法交互，因此对这个工具类进行了改进。
### MBProgressHUD(v1.1.0)
我主要写了一个分类，有菊花、文字、文字+图片提示，并提供了显示完成对调，方便显示完成后进行相应的操作或界面跳转，提供的调用方法如下：
```
#pragma mark ------ < Mask Layer > ------
#pragma mark
/** << 设置是否显示蒙层 > */
+ (void)wb_maskLayerEnabled:(BOOL)enabled;

#pragma mark --------  Basic Method  --------
#pragma mark
/**
 *  快速创建提示框 有菊花
 *
 *  @param message 提示信息
 *  @param view 显示视图
 *  @return hud
 */
+ (MBProgressHUD *)wb_showActivityMessage:(NSString *)message
                                   toView:(UIView *)view;
/**
 *  显示提示文字
 *
 *  @param message 提示信息
 *  @param view 显示的视图
 */
+ (void)wb_showMessage:(NSString *)message
                toView:(UIView *)view completion:(MBProgressHUDCompletionBlock)completion;
/**
 *  自定义成功提示
 *
 *  @param success 提示文字
 *  @param view 显示视图
 */
+ (void)wb_showSuccess:(NSString *)success
                toView:(UIView *)view completion:(MBProgressHUDCompletionBlock)completion;
/**
 *  自定义失败提示
 *
 *  @param error 提示文字
 *  @param view 显示视图
 */
+ (void)wb_showError:(NSString *)error
              toView:(UIView *)view completion:(MBProgressHUDCompletionBlock)completion;
/**
 *  自定义提示信息
 *
 *  @param info 提示信息
 *  @param view 示视图
 */
+ (void)wb_showInfo:(NSString *)info
             toView:(UIView *)view
         completion:(MBProgressHUDCompletionBlock)completion;

/**
 *  自定义警告提示
 *
 *  @param warning 提示信息
 *  @param view 示视图
 */
+ (void)wb_showWarning:(NSString *)warning
                toView:(UIView *)view completion:(MBProgressHUDCompletionBlock)completion;

/**
 *  自定义提示框
 *
 *  @param text 提示文字
 *  @param icon 图片名称
 *  @param view 展示视图
 */
+ (void)wb_show:(NSString *)text
           icon:(NSString *)icon
           view:(UIView *)view
     completion:(MBProgressHUDCompletionBlock)completion;

#pragma mark --------  Activity && Text  --------
#pragma mark
/**  < 只显示菊花 >  */
+ (MBProgressHUD *)wb_showActivity;
/**  < 菊花带有文字 >  */
+ (MBProgressHUD *)wb_showActivityMessage:(NSString *)message;

#pragma mark --------  Text && Image  --------
#pragma mark

/**
 文字提示

 @param message 提示文字
 @param completion 完成回调
 */
+ (void)wb_showMessage:(NSString *)message completion:(MBProgressHUDCompletionBlock)completion;

/**
 成功提示

 @param success 提示文字
 @param completion 完成回调
 */
+ (void)wb_showSuccess:(NSString *)success completion:(MBProgressHUDCompletionBlock)completion;

/**
 错误提示

 @param error 提示文字
 @param completion 完成回调
 */
+ (void)wb_showError:(NSString *)error completion:(MBProgressHUDCompletionBlock)completion;

/**
 信息提示

 @param info 提示文字
 @param completion 完成回调
 */
+ (void)wb_showInfo:(NSString *)info completion:(MBProgressHUDCompletionBlock)completion;

/**
 警告提示

 @param warning 提示文字
 @param completion 完成回调
 */
+ (void)wb_showWarning:(NSString *)warning completion:(MBProgressHUDCompletionBlock)completion;
```
举一个.m显示菊花方法的例子吧，MBProgressHUD最新版本对比老版本API还是有些变化的：
```
+ (MBProgressHUD *)wb_showActivityMessage:(NSString *)message
                                   toView:(UIView *)view {
    if (!view) view = [UIApplication sharedApplication].delegate.window;    /**  快速显示提示信息  */
    MBProgressHUD * hud = [MBProgressHUD showHUDAddedTo:view animated:YES];
    /**  < 显示动画效果 >  */
    hud.animationType = MBProgressHUDAnimationZoom;
    /**  < 文字内容 >  */
    hud.label.text = message;
    /**  < 影藏后移除视图 >  */
    hud.removeFromSuperViewOnHide = YES;
    /**  中间方框背景色  */
    hud.bezelView.color = [[UIColor blackColor] colorWithAlphaComponent:0.85f];
    /**  内容颜色  */
    hud.contentColor = [UIColor whiteColor];
    /**  < 最小显示时间 >  */
    hud.minShowTime = kActivityMinDismissTime;
    [self wb_configMaskLayer:hud];
    return hud;
}
```
具体详情，请戳：[WBMBProgressHUDManager](https://github.com/wenmobo/WBMBProgressHUDManager)
![Untitled.gif](https://user-gold-cdn.xitu.io/2018/2/2/16156e6d8ec2f0b5?w=312&h=612&f=gif&s=247767)
### 结语
> 选择哪一款提示框架，都要看自己喜好了，因为我代码水平有限，有些地方可能考虑的不够完善，只能说是抛砖引玉吧，大神们可能有更好的封装。要想基于这些框架自定义出自己需要风格的UI，还是要对框架提供的方法属性有一定的了解。