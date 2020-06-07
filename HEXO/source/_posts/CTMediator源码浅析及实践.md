---
title: CTMediator源码浅析及实践
tags: [iOS,CocoaPods,组件化,APP架构]
date: 2020-06-04 14:54:44
permalink:
categories: iOS
description: 组件化开发的概念已经出现很久了，当APP业务庞大到无法很好的团队协作开发，组件化开发模式就派上用场了。发展演变的今天，已经有很多解决方案了，最具有代表的就是**Target-Action**，**URL**，**Protocol**这三种方式，不管用哪一种，适合才是最好的。
image: https://tva1.sinaimg.cn/large/007S8ZIlly1gfgairj455j31ge0u0dos.jpg
---
<p class="description"></p>

<!-- more -->

## 前言

> 组件化开发的概念已经出现很久了，当APP业务庞大到无法很好的团队协作开发，组件化开发模式就派上用场了。发展演变的今天，已经有很多解决方案了，最具有代表的就是**Target-Action**，**URL**，**Protocol**这三种方式，不管用哪一种，适合才是最好的。
>
> 从自己接触到这个概念，就一直很感兴趣，但那会儿公司也没有强制要求用组件化开发，确实公司业务也没有达到需要用这种模式，盲目实行只会增加维护管理的成本。
>
> 最近在公司项目，基于**Target-Action**中间着模式，对现有的工程进行组件化改造，他是**casatwy**老师开源的一个组件化库，最近对源码进行了阅读，源码并不多，就一百来行，精简但是却很强大。下面对源码进行解读吧。

## 思维导图

![](https://tva1.sinaimg.cn/large/007S8ZIlly1gfgairj455j31ge0u0dos.jpg)

## 实现原理

### CTMediator

其实现原理是基于Runtime实现方法调度

```objective-c
- (id)performSelector:(SEL)aSelector withObject:(id)object;
```

```objective-c
NSInvocation
```

核心源码

```objective-c
- (id)safePerformAction:(SEL)action target:(NSObject *)target params:(NSDictionary *)params
{
    NSMethodSignature* methodSig = [target methodSignatureForSelector:action];
    if(methodSig == nil) {
        return nil;
    }
    const char* retType = [methodSig methodReturnType];

    if (strcmp(retType, @encode(void)) == 0) {
        NSInvocation *invocation = [NSInvocation invocationWithMethodSignature:methodSig];
        [invocation setArgument:&params atIndex:2];
        [invocation setSelector:action];
        [invocation setTarget:target];
        [invocation invoke];
        return nil;
    }

    if (strcmp(retType, @encode(NSInteger)) == 0) {
        NSInvocation *invocation = [NSInvocation invocationWithMethodSignature:methodSig];
        [invocation setArgument:&params atIndex:2];
        [invocation setSelector:action];
        [invocation setTarget:target];
        [invocation invoke];
        NSInteger result = 0;
        [invocation getReturnValue:&result];
        return @(result);
    }

    if (strcmp(retType, @encode(BOOL)) == 0) {
        NSInvocation *invocation = [NSInvocation invocationWithMethodSignature:methodSig];
        [invocation setArgument:&params atIndex:2];
        [invocation setSelector:action];
        [invocation setTarget:target];
        [invocation invoke];
        BOOL result = 0;
        [invocation getReturnValue:&result];
        return @(result);
    }

    if (strcmp(retType, @encode(CGFloat)) == 0) {
        NSInvocation *invocation = [NSInvocation invocationWithMethodSignature:methodSig];
        [invocation setArgument:&params atIndex:2];
        [invocation setSelector:action];
        [invocation setTarget:target];
        [invocation invoke];
        CGFloat result = 0;
        [invocation getReturnValue:&result];
        return @(result);
    }

    if (strcmp(retType, @encode(NSUInteger)) == 0) {
        NSInvocation *invocation = [NSInvocation invocationWithMethodSignature:methodSig];
        [invocation setArgument:&params atIndex:2];
        [invocation setSelector:action];
        [invocation setTarget:target];
        [invocation invoke];
        NSUInteger result = 0;
        [invocation getReturnValue:&result];
        return @(result);
    }

#pragma clang diagnostic push
#pragma clang diagnostic ignored "-Warc-performSelector-leaks"
    return [target performSelector:action withObject:params];
#pragma clang diagnostic pop
}
```

头文件开放的两个接口方法，都是对上面这个方法传入参数进行处理，包括一些容错处理

```objective-c
// 远程App调用入口
- (id _Nullable)performActionWithUrl:(NSURL * _Nullable)url completion:(void(^_Nullable)(NSDictionary * _Nullable info))completion;
// 本地组件调用入口
- (id _Nullable )performTarget:(NSString * _Nullable)targetName action:(NSString * _Nullable)actionName params:(NSDictionary * _Nullable)params shouldCacheTarget:(BOOL)shouldCacheTarget;
```

### CTMediator+HandyTools

CTMediator的扩展方法，可以不再VC类里实现跳转，减少依赖，实现解耦。

```objective-c
- (UIViewController * _Nullable)topViewController;
- (void)pushViewController:(UIViewController *)viewController animated:(BOOL)animated;
- (void)presentViewController:(UIViewController *)viewControllerToPresent animated:(BOOL)animated completion:(void (^ _Nullable )(void))completion;
```

## 实际应用

### 一开始就实施

如果一开始都采用这种架构模式，实现组件化开发还是很容易的，可以更多的专注于业务开发。

### 在现有工程实施

如果是对现有工程实施改造，如果之前模块划分混乱，那么改造起来是相当的痛苦，各种依赖，剪不断，理还乱。

- 第一步，制作基础依赖库，供上层业务依赖，如网络请求，工具类，基础UI组件，比较稳定，变动较少
- 第二步，拆分业务模块，跳转实施改造，创建Target_XX，文件，XX是targetName，**nativeFetchDetailViewController**是actionName

```objective-c
- (UIViewController *)Action_nativeFetchDetailViewController:(NSDictionary *)params
{
    // 因为action是从属于ModuleA的，所以action直接可以使用ModuleA里的所有声明
    DemoModuleADetailViewController *viewController = [[DemoModuleADetailViewController alloc] init];
    viewController.valueLabel.text = params[@"key"];
    return viewController;
}
```

创建**CTMediator (CTMediatorModuleAActions)**分类，添加跳转方法，这个文件就是需要维护的依赖文件，实现依赖解耦

```objective-c
- (UIViewController *)CTMediator_viewControllerForDetail;
```

- 第三步，解决编译报错，并集成到主工程，让主工程编译通过

当然，CTMediator主要通过Runtime方法调度，不仅仅是实现跳转，它可以动态调度**Target_XX**任意返回值得方法，如基础数据类型，void，OC的类，从而达到解耦的目的，在调用的时候，可以传参是否缓存Target，这在cell复用的时候非常有用，在VC销毁时清除Target即可。

## 结语

> 相对于一般的开源库，**casatwy**老师的这个库，代码非常的精简，但却非常实用强大，可看出作者优秀的架构设计能力。代码不在多，而在精。对于我们日常开发也是很有启发的，对于一个新功能的开发，在开发之前，是不是应该考虑到可扩展，可复用，稳定性，实用性，然后才进行开发。组件化的实施，对于业务多，团队开发，还是很有必要的，但实施工程也会有各种各样的问题，需要不断优化解决，才能让APP更加稳定，更加易于维护。

<hr />
