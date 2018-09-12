---
title: CocoaPods进阶：制作公有库
tags: [CocoaPods,GitHub,MBProgressHUD,SVProgressHUD,iOS]
date: 2018-09-03 21:35:29
permalink:
categories: iOS
description: 在CocoaPods进阶：详解私有库制作这篇博客中，讲解记录了cocoapods使用pod lib create [projectname]命令模板化创建pod私有库，这篇博客主要讲解cocoapods制作共有库的过程。这里，我不在使用模板化方式创建，而是用原来在GitHub上已经提交过的项目（如果你舍不得获得的star，最好采用这种方式啦，😆）。自己也是参照博客资料，本来以为自己有了制作私有库的基础，制作共有库会没那么多的坑，但自己还是折腾了一晚上，好了，不多说了，开始讲解共有库的制作吧！
image:
---
<p class="description"></p>

<!-- more -->

### 一、 前言

在[CocoaPods进阶：详解私有库制作](http://blogwenbo.com/2018/08/13/CocoaPods%E8%BF%9B%E9%98%B6%EF%BC%9A%E8%AF%A6%E8%A7%A3%E7%A7%81%E6%9C%89%E5%BA%93%E5%88%B6%E4%BD%9C/)这篇博客中，讲解记录了cocoapods使用`pod lib create [projectname]`命令模板化创建pod私有库，这篇博客主要讲解cocoapods制作共有库的过程。这里，我不在使用模板化方式创建，而是用原来在GitHub上已经提交过的项目（如果你舍不得获得的star，最好采用这种方式啦，😆）。自己也是参照博客资料，本来以为自己有了制作私有库的基础，制作共有库会没那么多的坑，但自己还是折腾了一晚上，好了，不多说了，开始讲解共有库的制作吧！

### 二、目录

> 1、创建spec文件
>
> 2、编辑podspec文件
>
> 3、本地库验证
>
> 4、推送打标签
>
> 5、验证podspec文件
>
> 6、注册，推送podspec到cocoapods，搜索验证

### 三、具体步骤
**1、创建spec文件**
在`xxxx.xcodeproj`同级目录下，创建podspec文件

```
pod spec create WBHUDManager
```

创建成功之后如下：

![](https://ws1.sinaimg.cn/large/0069RVTdly1fuwp01rvz4j30ei0e8q59.jpg)

**注意**：

这里新创建的**podspec**最好要和**LICENSE**、**README.md**在同级目录，自己在这里也折腾了许久，头一次用非模板方式，也踩了不少的坑。这里一定要注意哦。

**2、编辑podspec文件****

用终端或者记事本编辑podspec文件，下面是我配置**WBHUDManager.podspec**，这里我也不做详细的介绍了，在我讲解的私有库制作博客，有对每个属性的详细描述，在网上也能轻易的查阅到相关的资料。

```ruby
Pod::Spec.new do |s|
  s.name             = 'WBHUDManager'
  s.version          = '1.0.0'
  s.summary          = 'iOS 基于SVProgressHUD、MBProgressHUD提示框封装'
  s.homepage         = 'https://github.com/wenmobo/WBHUDManager'
  s.license          = { :type => 'MIT', :file => 'LICENSE' }
  s.author           = { 'wenmobo' => 'wenmobo2018@gmail.com' }
  s.source           = { :git => 'https://github.com/wenmobo/WBHUDManager.git', :tag => s.version.to_s }
  s.ios.deployment_target = '8.0'
  s.requires_arc = true
  s.source_files =  'WBHUDManager/WBHUDManager.h'
  s.frameworks = 'UIKit'
  
  s.subspec 'SVProgressHUDWBAddtional' do |ss|
      ss.source_files = 'WBHUDManager/SVProgressHUDWBAddtional/*.{h,m}'
      ss.dependency 'SVProgressHUD'
  end
  
  s.subspec 'MBProgressHUDWBAddtional' do |ss|
      ss.source_files = 'WBHUDManager/MBProgressHUDWBAddtional/*.{h,m}'
      ss.resource = 'WBHUDManager/MBProgressHUDWBAddtional/MBProgressHUD.bundle'
      ss.dependency 'MBProgressHUD'
  end

end
```

**3、本地库验证**

```
pod lib lint WBHUDManager.podspec
```

如果有警告，需要根据提示内容解决警告，忽略警告

```
pod lib lint WBHUDManager.podspec --allow-warnings
```

**4、推送打标签**

由于这里我之前已经推送到远程了，所以只需要打标签就可了，注意要和WBHUDManager.podspec中version保持一致：

```
git tag -m 'release version 1.0.0' 1.0.0

git push origin 1.0.0 
或者
git push --tags
```

**5、验证podspec文件**

推送标签之后，需对WBHUDManager.podspec进行验证

```
pod spec lint WBHUDManager.podspec
```

**6、注册，推送podspec到cocoapods，验证**

验证通过之后，需要使用邮箱注册cocoapods，终端输入：

```
pod trunk register [email] ‘用户名’ --description='MacBook Pro'

example
pod trunk register 123@qq.com 'wenbo' --description='MacBook Pro'
```

之后会给你发送一条邮箱，进行确认，这里的-**-description='MacBook Pro'**可以省略

查看个人信息

```
pod trunk me
```

![](https://ws3.sinaimg.cn/large/0069RVTdly1fuwpleoxwyj30vk074wfw.jpg)

推送podspec到**cocoapods**

```
pod trunk push WBHUDManager.podspec
```

推送成功之后，终端输出如下

![](https://ws1.sinaimg.cn/large/0069RVTdly1fuwpojsca1j30v60mcq70.jpg)

之后我们可以搜索验证

```
pod search WBHUDManager
```

![](https://ws1.sinaimg.cn/large/0069RVTdly1fuwpq0w8uvj30vq0awtau.jpg)

哈哈，已经发布成功啦，是不是很开心啦☺️。

### 四、问题解决

- 发布成功之后搜索不到

- ```
  //删除本地索引
  rm ~/Library/Caches/CocoaPods/search_index.json
  
  //搜索
  pod search [库名]
  
  //更新索引
  pod repo update
  ```

### 五、打广告

> 哈哈，最后也为自己打波广告吧，这篇博客使用的例子是自己基于[MBProgressHUD](https://github.com/jdg/MBProgressHUD)、[SVProgressHUD](https://github.com/SVProgressHUD/SVProgressHUD)封装的一个提示框架，支持超多自定义属性设置，HUD的状态切换，显示完成回调，这也是我发布的第一个公有库，如果有写的不好的地方，请多多包涵。喜欢的朋友记得**star**鼓励下哟，最后贴出GitHub地址吧：

[WBHUDManager](https://github.com/wenmobo/WBHUDManager)