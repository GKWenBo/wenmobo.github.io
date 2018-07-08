---
title: Chisel-LLDB调试助手
tags: [iOS,LLDB,Terminal,Chisel]
date: 2018-07-08 11:52:01
permalink:
categories: iOS
description: 在我查阅如何定位视图约束冲突相关博客资料的时候，了解到了**Facebook**开源的这款LLDB调试工具，通过Chisel可以轻松的找到约束有冲突的视图，自己以前只接触过Xcode自带LLDB工具，用的也不算太多，通过阅读相关文档，发现**Chisel**有很多实用的功能，比如说打印视图层级关系，通过命令添加断点，打印对象继承关系，预览打开UIImage、CGImageRef图片，显示隐藏视图或layer等等
image:
---
<p class="description"></p>

<!-- more -->

> Chisel is a collection of LLDB commands to assist debugging iOS apps.

### 目录
> - GitHub地址
> - 安装
> - 常用常用Commands
> - 推荐博客
### 一、GitHub地址
[Chisel](https://github.com/facebook/chisel)
### 二、安装
- 未安装Homrebrew，先安装Homrebrew
  参考[MAC上Homebrew常用命令](https://www.jianshu.com/p/c60789934af1)
- 安装Chisel
```
brew install chisel
```
- 如果没有创建**.lldbinit**文件，则在终端创建文件
```
touch .lldbinit 
//open .lldbinit
```
- 编辑**.lldbinit**文件，并添加以下内容`command script import /usr/local/opt/chisel/libexec/fblldb.py`
```
vim .lldbinit

//添加以下内容
# ~/.lldbinit
...
command script import /path/to/fblldb.py
```
最后wq保存，重启Xcode，就可以使用Chisel了。
### 三、常用Commands
| 命令            | 命令描述                                                     | iOS  | OS X |
| :-------------- | :----------------------------------------------------------- | :--- | :--- |
| pviews          | Print the recursive view description for the key window.     | YES  | YES  |
| pvc             | Print the recursive view controller description for the key window. | YES  | NO   |
| visualize       | Open a UIImage, CGImageRef, UIView, CALayer, NSData (of an image), UIColor, CIColor, or CGColorRef in Preview.app on your Mac. | YES  | NO   |
| fv              | Find a view in the hierarchy whose class name matches the provided regex. | YES  | NO   |
| fvc             | Find a view controller in the hierarchy whose class name matches the provided regex. | YES  | NO   |
| show/hide       | Show or hide the given view or layer. You don't even have to continue the process to see the changes! | YES  | YES  |
| mask/unmask     | Overlay a view or layer with a transparent rectangle to visualize where it is. | YES  | NO   |
| border/unborder | Add a border to a view or layer to visualize where it is.    | YES  | YES  |
| caflush         | Flush the render server (equivalent to a "repaint" if no animations are in-flight). | YES  | YES  |
| bmessage        | Set a symbolic breakpoint on the method of a class or the method of an instance without worrying which class in the hierarchy actually implements the method. | YES  | YES  |
| wivar           | Set a watchpoint on an instance variable of an object.       | YES  | YES  |
| presponder      | Print the responder chain starting from the given object.    | YES  | YES  |
| ...             | ...                                                          | ...  | ...  |
就介绍这么多了，现在自己用到的也并不算太多，做下记录，方便自己以后查阅，有兴趣的朋友可以自行了解其用法吧。
### 推荐博客
1、[LLdb篇2教你使用faceBook的chisel来提高调试效率](https://www.jianshu.com/p/b2371dd4443b)  
2、[Chisel-LLDB命令插件，让调试更Easy](https://blog.cnbluebox.com/blog/2015/03/05/chisel/)