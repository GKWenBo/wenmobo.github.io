---
title: CocoaPods进阶：详解私有库制作
tags: [CocoaPods,iOS,GitHub,Git]
date: 2018-08-13 21:58:51
permalink:
categories: iOS
description: 自己一直想用CocoaPods制作pod库，在自己面试过程中也被面试官问到过组件化开发的概念，然尔自己那时也不是很了解，CocoaPods与组件化也息息相关，利用CocoaPods也便于维护pod组件，于是自己就决定学习用CocoaPods制作pod库，下面就开始讲解私有库的制作过程吧。
image: https://ws4.sinaimg.cn/large/006tNbRwly1funipf7wcfj31kw0s441m.jpg
---
<p class="description"></p>

<!-- more -->

### 前言

> 自己一直想用CocoaPods制作pod库，在自己面试过程中也被面试官问到过组件化开发的概念，然尔自己那时也不是很了解，CocoaPods与组件化也息息相关，利用CocoaPods也便于维护pod组件，于是自己就决定学习用CocoaPods制作pod库，下面就开始讲解私有库的制作过程吧。

### 目录

- 安装CocoaPods
- 创建远程内部私有Spec Repo仓库
- 模板创建pod库
- 编辑***.podspec文件
- 验证本地是否通过
- 关联本地仓库，并推送到远程仓库，打标签
- 推送***.podspec到远程
- 验证远程是否通过
- 验证私有仓库是否可用，pod集成私有库

#### 安装CocoaPods

首先要安装CocoaPods，没有安装可以参考我的博客[CocoaPods安装与使用](https://www.jianshu.com/p/f218fe3baff8)。

#### 创建远程内部私有Spec Repo仓库

这步自己采坑不少，一开始自己并不理解，不知到代码仓库和Spec Repo是需要分开存储的。好了，不说自己经历的曲折了，如果你还没有创建远程内部私有Spec Repo仓库, 需要到[Github](https://github.com/),[码云](http://git.oschina.net/)或其他代码托管平台创建远程仓库, 之后将远程仓库克隆到本地，终端执行如下命令：

```
//这里可以用https或ssh地址方式克隆
pod repo add WBSpecs git@github.com:wenmobo/WBSpecs.git
```

克隆之后，本地cocoapods目录如下：

![](https://ws4.sinaimg.cn/large/0069RVTdly1fuaqlqavaij31js054dhh.jpg)

#### 模板创建pod库

- **第二步**：在本地任意一个文件夹下创建pod库：

  ```
  pod lib create WBAvoidCrash
  ```

  之后控制台输出

  ![](https://ws2.sinaimg.cn/large/006tNbRwly1fu8fsp6s32j30vo05kmya.jpg)

  接着会需要回答一些问题：

  ```
  # 你想使用哪个平台？
  1、What platform do you want to use?? [ iOS / macOS ]
  iOS
  # 库语言选择？
  2、What language do you want to use?? [ Swift / ObjC ]
  ObjC
  # 你要使用哪个测试框架？
  3、Which testing frameworks will you use? [ Specta / Kiwi / None ]
  None
  # 是否要UI测试？
  4、Would you like to do view based testing? [ Yes / No ]
  NO
  # 类名前缀？
  5、What is your class prefix?
  WB
  ```

  成功之后，目录如下：

  ![](https://ws3.sinaimg.cn/large/006tNbRwly1fu8g2rfjkhj30m00cm768.jpg)

  工程目录如下：

  ![](https://ws3.sinaimg.cn/large/006tNbRwly1fu8g5h5i4ej31js13c4gs.jpg)

- 在工程**WBAvoidCrash**目录添加我们的代码文件：

  ![](https://ws4.sinaimg.cn/large/006tNbRwly1fu8gebt4mpj30f00bgwg2.jpg)

  添加完成之后如下：

  ![](https://ws1.sinaimg.cn/large/006tNbRwly1fu8ghg3iwbj30eg0v0100.jpg)

  **注意**：代码文件需要添加到**WBAvoidCrash/Classes**目录下。

#### 编辑***.podspec文件

```
Pod::Spec.new do |s|
  #库名称
  s.name             = 'WBAvoidCrash'
  
  #指定支持的平台和版本，不写则默认支持所有的平台，如果支持多个平台，则使用下面的deployment_target定义
  spec.platform = :ios
  
  #版本号
  s.version          = '1.0.0'
  
  #库简短介绍
  s.summary          = 'iOS 防Crash库'
  
  #开源库描述 
  s.description      = <<-DESC
TODO: iOS防crash库分类.
                       DESC
                       
  #开源库地址，或者是博客、社交地址等
  s.homepage         = 'https://github.com/wenmobo/WBAvoidCrash'
  
  #开源协议
  s.license          = { :type => 'MIT', :file => 'LICENSE' }
  
  #开源库作者
  s.author           = { 'wenmobo' => 'wenmobo2018@gmail.com' }
  
  #开源库GitHub的路径与tag值，GitHub路径后必须有.git,tag实际就是上面的版本
  s.source           = { :git => 'https://github.com/wenmobo/WBAvoidCrash.git', :tag => s.version }
  
  #社交网址
  s.social_media_url = 'http://blogwenbo.com/'
  
  #开源库最低支持
  s.ios.deployment_target = '8.0'
  
  #源库资源文件
  s.source_files = 'WBAvoidCrash/Classes/**/*'
  
  #是否支持arc
  s.requires_arc = true
  
  #依赖系统库
  s.frameworks = 'Foundation'
  
  #开源库依赖库
  # s.dependency "Masonry", "~> 1.0"
  
  #添加系统依赖静态库
  #s.library = 'sqlite3', 'xml2'
  
  #添加依赖第三方的framework
  #s.vendored_frameworks = 'XXXX/XXXX/**/*.framework'
  
  #静态库.a
  s.vendored_library = 'XXXX/XXX/XXX.a', 'YYY/YYY/Y.a'
  
  #添加资源文件
  #s.resource = 'XXX/XXXX/**/*.bundle'
  
  #在 podspec 文件中添加 s.static_framework = true，CocoaPods 就会把这个库配置成static framework。同时支持 Swift 和 Objective-C
  #s.static_framework = true
end
```

- **关于s.source_files写法**

```
//表示匹配WBAvoidCrash/Classes下所有文件(主目录和子目录，其中**相当于省略中间层级)
'WBAvoidCrash/Classes/**/*'
//表示匹配Classes所有以.h和.m结尾的文件
'WBAvoidCrash/Classes/*.{h,m}'
//表示匹配所有WBAvoidCrash目录下文件，不包含子目录
'WBAvoidCrash/*'
```

更多关于资源目录层级写法可以参考GitHub一些著名框架，[AFNetworking.podspec](https://github.com/AFNetworking/AFNetworking/blob/master/AFNetworking.podspec)、[ZFPlayer.podspec](https://github.com/renzifeng/ZFPlayer/blob/master/ZFPlayer.podspec)等。

- **s.dependency关于依赖三库，依赖多个三方库如下：**

- ```
  s.dependency 'Masonry'
  s.dependency 'MJRefresh'
  s.dependency Masonry 'YYModel'
  ```

#### 验证本地是否通过

- 配置好podspec之后，验证本地库是否通过验证，终端输入如下命令：

```
pod lib lint
```

通过验证，终端输出如下：

![](https://ws2.sinaimg.cn/large/0069RVTdly1fuarsoxm53j30vi05ijso.jpg)

- 报如下错误

![屏幕快照 2018-08-14 上午12.21.07.png](https://upload-images.jianshu.io/upload_images/3072214-258e21287c18f67b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

需在Xcode中配置：

![屏幕快照 2018-08-14 上午12.34.17.png](https://upload-images.jianshu.io/upload_images/3072214-416eef1ea9e99431.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 如果pod库存在警告是不能通过验证的，如果要暂时忽略警告通过验证（如码云创建的私有库**s.homepage**地址不可达警告），可使用如下命令：

```
pod lib lint --allow-warnings
```

- 你制作的pod库依赖三方库，而三方库包含静态库（如：**xxxx.a**），在验证的时候，不能验证通过，可使用如下命令：

- ```
  pod lib lint --use-libraries
  
  //同时忽略警告
  pod lib lint --use-libraries --allow-warnings
  ```

不管怎样都要解决pod库存在的警告，并通过验证。

#### 关联本地仓库，并推送到远程仓库，打标签

- 如果你还未创建远程仓库，你需要创建与之对应的远程仓库，我是在GitHub创建的仓库，这里也不再赘述创建方法。创建之后须与本地仓库关联，在终端执行如下命令：

```
#提交代码到暂存区
git add .
#提交到本地仓库
git commit -m "create WBAvoidCrash Library"
#添加到远程仓库
git remote add origin git@github.com:wenmobo/WBAvoidCrash.git
#推送到远程仓库
git push origin master
```

- 最近在用码云制作私有库的时候按照上面git命令，在执行`git push origin master`会报错，需要执行以下命令或者按终端提示的信息操作，第一次才能成功推送到远程仓库：

- ```
  git pull --rebase origin master
  ```

- 提交完成之后进行打标签操作：

```
#打标签
git tag -a 1.0.0 -m 'release version 1.0.0'
#推送标签到远程
git push origin 1.0.0
```

**友情提示**

关于git打标签操作，你可以借助[*Sourcetree*](https://www.sourcetreeapp.com/)或者终端命令，可以查看我的博客[MAC上Git打标签](https://www.jianshu.com/p/3e85e15c5e43)。

#### 推送***.podspec到远程

首先将本地WBAvoidCrash.podspec推送到远程私有repo spec仓库和本地repo spec仓库，终端执行如下命令：

```
cd [WBAvoidCrash库路径]
pod repo push WBSpecs WBAvoidCrash.podspec
```

![](https://ws4.sinaimg.cn/large/0069RVTdly1fuat1gb99mj30y00eijth.jpg)

#### 验证远程是否通过

推送成功之后，终端输入如下命令进行验证：

```
pod spec lint WBAvoidCrash.podspec
```

验证通过终端输出如下：

![](https://ws3.sinaimg.cn/large/0069RVTdly1fuat21dm07j30ye08e3zt.jpg)

- 同样这里如果还存在着警告或者错误，同样不能验证通过，同样可以用以下命令忽略警告通过验证：

  ```
  pod spec lint WBAvoidCrash.podspec --allow-warnings
  pod spec lint WBAvoidCrash.podspec --use-libraries
  pod spec lint WBAvoidCrash.podspec --allow-warnings --use-libraries
  ```

#### 验证私有仓库是否可用，pod集成私有库

验证通过之后，下面进行测试，看是否能通过cocoapods集成到我们的项目，首先用pod命令进行搜索，看能否搜索到：

```
pod search WBAvoidCrash
```

这时可能会报如下错误

![](https://ws3.sinaimg.cn/large/0069RVTdly1fuat8rv62bj30xw04s3zw.jpg)

不要慌，在终端执行如下命令，然后重新search：

```
rm ~/Library/Caches/CocoaPods/search_index.json
```

耐心等待之后，发现能搜到自己创建的私有库了：

![](https://ws1.sinaimg.cn/large/0069RVTdly1fuatj9r8cwj30xu07agmu.jpg)

新建一个测试工程测试，用CocoaPods初始化项目，编辑**podfile**文件：

```shell
#CocoaPods官方spec仓库
source 'https://github.com/CocoaPods/Specs.git'
#自己私有spec仓库
source 'https://github.com/wenmobo/WBSpecs.git'

platform :ios, '8.0'

target 'Test' do
  #防Crash库
  pod 'WBAvoidCrash'

  # Pods for Test

  target 'TestTests' do
    inherit! :search_paths
    # Pods for testing
  end

  target 'TestUITests' do
    inherit! :search_paths
    # Pods for testing
  end

end

```

编辑好**podfile**文件之后，终端执行：

```
pod install
或
pod install --no-repo-update
```

耐心等待一会儿，你会发现私有库已经集成到测试项目中了：

![](https://ws3.sinaimg.cn/large/006tNbRwly1fung83adapj31jy0t8k1k.jpg)

到这里，cocoapods私有库制作已讲解完成。也为自己制作的第一个私有库打波广告吧☺️☺️，[WBAvoidCrash](https://github.com/wenmobo/WBAvoidCrash)一个防Crash库，现在支持9种防崩溃类型，集成方便，使用无需导入相关的头文件，这个库之前没这么完善，后来参考借鉴了一些大神开源的库。下面👇贴出私有库地址吧：

[WBAvoidCrash](https://github.com/wenmobo/WBAvoidCrash)

### 相关命令

#### cocoapods

- 更新repo

  ```
  //更新所有repo
  pod repo update
  
  pod repo [spec库名]
  ```

- 验证pod库

  ```
  //验证本地pod库
  pod lib lint
  //本地验证忽略警告
  pod lib lint --allow-warnings
  
  //验证远程
  pod spec lint [name].podspec
  ```

- 搜索pod库

  ```
  pod search [库名]
  ```

### 结语

> 终于完成这篇博客了，从自己比较熟悉GitHub之后，也想过自己能够开源一款三方库，然而自己水平有限，现在还没有拿的出来好的封装库或一些好的封装思想。但自己还是要学会制作pod库，在写博客之前，自己在谷歌浏览器查了许多的资料，资料也是比较的凌乱，自己在制作过程中也踩了许多的坑，最后自己也成功制作了一个私有库[WBAvoidCrash](https://github.com/wenmobo/WBAvoidCrash)，过程虽然有些坎坷，但自己还是很有成就感。自己也是第一次制作，如果有描述不对的地方，希望大家能够批评指正，我也会第一时间修改，同时也希望这篇博客对需要的朋友一些帮助，接下来我也会写一篇记录公开库制作过程的博客。

### 参考文章
1、[ CocoaPods创建公有和私有Pod库方法总结](https://segmentfault.com/a/1190000007947371)
2、[出现Unable to find a pod with name, author, summary, or description matching解决方法](https://blog.csdn.net/conglin1991/article/details/55096422)
3、[如何发布自己的开源框架到CocoaPods](https://www.jianshu.com/p/d2d81b58d716)
4、[使用CocoaPods管理iOS库---制作pod篇](https://www.jianshu.com/p/d2d81b58d716)
5、[如何创建私有 CocoaPods 仓库](https://www.jianshu.com/p/ddc2490bff9f)
6、[Making a CocoaPod](https://guides.cocoapods.org/making/making-a-cocoapod.html)
7、[Create and Distribute Private Libraries with Cocoapods](https://medium.com/@shahabejaz/create-and-distribute-private-libraries-with-cocoapods-5b6507b57a03)