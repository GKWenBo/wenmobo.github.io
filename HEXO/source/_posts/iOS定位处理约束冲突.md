---
title: iOS定位处理约束冲突
tags: [iOS,Objective-C,Swift,Xcode,Chisel]
date: 2018-07-08 11:58:08
permalink:
categories: iOS
description: 在做项目的时候，无意间看到自动布局约束警告，开始也也觉得没什么，虽然有警告，但并不影响UI展示效果。但是越来越有代码洁癖的我，也忍受不了控制台输出一大堆约束警告Log，于是就查阅如何定位解决约束冲突，同时自己也记录下来。下面开始介绍具体操作步骤吧。
image:
---
<p class="description"></p>

<!-- more -->

### 一、添加UIViewAlertForUnsatisfiableConstraints断点
- 添加**Symbolic Breakpoints**
  ![屏幕快照 2018-06-04 下午2.44.35.png](https://user-gold-cdn.xitu.io/2018/6/10/163e857ace3a0af6?w=428&h=226&f=png&s=37641)
- 编辑断点symbol填入
```
UIViewAlertForUnsatisfiableConstraints
```
- 添加控制台打印action
``` 
po [[UIWindow keyWindow] _autolayoutTrace]
```
![屏幕快照 2018-06-04 下午2.44.50.png](https://user-gold-cdn.xitu.io/2018/6/10/163e857ace93dcbb?w=998&h=514&f=png&s=129745)
### 二、定位约束警告冲突
- 添加好断点之后，当界面有约束冲突，就会触发断点，控制打印如下：
```
[LayoutConstraints] Unable to simultaneously satisfy constraints.
	Probably at least one of the constraints in the following list is one you don't want. 
	Try this: 
		(1) look at each constraint and try to figure out which you don't expect; 
		(2) find the code that added the unwanted constraint or constraints and fix it. 
(
	<MASLayoutConstraint:0x604000ab04a0 UIButton:0x7faf99f04010.width == 40>,
	<MASLayoutConstraint:0x604000ab66e0 UIButton:0x7faf99f04010.left == CYBButtonView:0x7faf99f83360.left + 10>,
	<MASLayoutConstraint:0x604000abaa00 UILabel:0x7faf99f5f8e0.left == UIButton:0x7faf99f04010.right>,
	<MASLayoutConstraint:0x604000abd580 UILabel:0x7faf99f5f8e0.left == CYBButtonView:0x7faf99f83360.left + 15>,
)

Will attempt to recover by breaking constraint 
<MASLayoutConstraint:0x604000ab04a0 UIButton:0x7faf99f04010.width == 40>

Make a symbolic breakpoint at UIViewAlertForUnsatisfiableConstraints to catch this in the debugger.
The methods in the UIConstraintBasedLayoutDebugging category on UIView listed in <UIKit/UIView.h> may also be helpful.
```
- 根据提示，找到约束有问题的控件地址**0x7faf99f04010**，然后全局搜索，就能找到具体是哪个控件
- 如果控制台打印不够直观看出是哪个控件约束有问题，我们可以通过 LLDB命令工具[chisel](https://github.com/facebook/chisel)定位寻找。

### 三、解决冲突
通常解决冲突的方法有：
> - 删除多余约束
> - 修改约束优先级
### 参考
1、[How to trap on UIViewAlertForUnsatisfiableConstraints?](https://stackoverflow.com/questions/26389273/how-to-trap-on-uiviewalertforunsatisfiableconstraints)