---
title: iOS APP瘦身实践
tags: [性能优化,iOS]
date: 2018-11-04 17:51:54
permalink:
categories: iOS
description: 好久没有更新博客了，自己也一直想写一些性能优化，APP瘦身的文章，现在就做下记录与分享，方便以后查阅。公司这段新需求不算太多，自己就抽时间，对APP安装包大小做了一些优化，优化后的成果还是很明显的，从之前的90M左右缩减到了40几M。网上还是很容易找到相关的文章，下面我总结下可以从哪几方面去做优化工作吧！
image:
---
<p class="description"></p>

<!-- more -->

### 一、前言

> 好久没有更新博客了，自己也一直想写一些性能优化，APP瘦身的文章，现在就做下记录与分享，方便以后查阅。公司这段新需求不算太多，自己就抽时间，对APP安装包大小做了一些优化，优化后的成果还是很明显的，从之前的90M左右缩减到了40几M。网上还是很容易找到相关的文章，下面我总结下可以从哪几方面去做优化工作吧！

### 二、图片资源优化

- 不看不知道，一看吓一跳，项目中的图片资源原来是如此占用大小，动不动一张图片就是好几兆，并且有很多还是没有用到的。找出项目中未用到的图片资源我们可以借助这款软件：

   [LSUnusedResources](https://github.com/tinymind/LSUnusedResources/raw/master/Release/LSUnusedResources.app.zip)

  借助着款开源软件我们可以找出项目中没有用到的

  ```
  [imageset, jpg, png, gif]
  ```

  当然，搜索出来的结果不一定是百分百准确的，还是要结合实际情况。

- 除了删除没有用到的图片资源，对于用到的大图片资源，我们还需要对大图进行压缩，压缩的同时还需要保证图片的显示效果，推荐一个非常棒的图片压缩网站吧：

  [tinypng](https://tinypng.com/)

  当然还有其他的压缩工具网站，可以自己去搜索探寻吧。

### 三、删除项目没有使用到的类

- 删除项目中未用到的类，就需要看经验了，没有用到的类，一般没有初始化相关的代码`alloc init`或`new`，这样就可以删除项目中未使用的类。这个工作最好是定期进行一次吧，如果等到很久来处理，那工作量真的就太大了，如果是需求变更频繁的话，肯定会产生很多未用到的类。所以还是定期做一次排查吧，以免留下沉重的历史包袱。
- 除了手工排查，还可以借助命令行工具 `find`命令，之前看到过一篇博客介绍，一时想不起来，等想起来在做补充吧。

### 四、LinkMap文件分析

- 关于LinkMap的介绍，网上一搜一大堆，这里就不做详细介绍了，说下怎么配置Xcode获得LinkMap文件和借助工具分析文件大小吧。

- XCode -> Project -> Build Settings -> 搜map -> 把Write Link Map File选项设为YES，并指定好linkMap的存储位置

- 编译后，到编译目录里找到该txt文件，目录如下：

  ```
  ~/Library/Developer/Xcode/DerivedData/XXX-XXXXXXXXXXXX/Build/Intermediates/XXX.build/Debug-iphoneos/XXX.build/
  ```

- 之后借助GitHub上[LinkMap](https://github.com/huanxsd/LinkMap)工具或[linkmap.js](https://gist.github.com/bang590/8f3e9704f1c2661836cd)，分析工程可执行文件大小。

### 五、移除模拟器支持文件

- 我们在集成一些三方库的时候，这些三方库可能包含模拟器指令集。比如说环信IMSDK，打包上传App Store需移除模拟器支持指令集。

- 查看静态库支持的指令集

  ```
  lipo -info libname.a(或者libname.framework/libname)
  ```

- 合并静态库

  ```
  lipo -create 静态库存放路径1  静态库存放路径2 ...  -output 整合后存放的路径
  lipo  -create  libname-armv7.a   libname-armv7s.a   libname-i386.a  -output  libname.a
  ```

- 静态库拆分

  ```
  lipo 静态库源文件路径 -thin CPU架构名称 -output 拆分后文件存放路径
  ```

### 参考文章

- [iOS APP安装包瘦身实践](https://www.jianshu.com/p/c94dedef90b7)
- [LinkMap文件分析](https://www.jianshu.com/p/4bd6d1315104)
- [APP瘦身 - 之framework](https://www.jianshu.com/p/2d0ba1bc2a50)