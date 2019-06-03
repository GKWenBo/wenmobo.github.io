---
title: iOS持续集成与自动打包
tags: [iOS,fastlane,Jenkins,fir.im,蒲公英]
date: 2018-12-23 23:36:03
permalink:
categories: iOS
description: 从事iOS开发也有一段时间了，实习的时候自己就了解了iOS打包分发的一些方式，自己也写了一篇博客[iOS打测试包与分发测试](https://www.jianshu.com/p/fc6721cf5c7f),介绍了如何打测试包以及上传相关的分发平台，也提到了脚本自动打包。现在自己负责几个项目的维护与开发工作，发现打一个包出来到上传到iTunes Connect上实在是太慢了，步骤也多，比较的耗时间，也不够自动化。后面自己了解了Jenkins持续构建工具，和fastlane自动打包工具，能够提高打包效率，下面开始介绍如何使用这些工具吧。
image: https://ws1.sinaimg.cn/large/006tNbRwly1fygs2adp2jj31ub0u0k95.jpg
---
<p class="description"></p>



<!-- more -->

# 前言
> 从事iOS开发也有一段时间了，实习的时候自己就了解了iOS打包分发的一些方式，自己也写了一篇博客[iOS打测试包与分发测试](https://www.jianshu.com/p/fc6721cf5c7f),介绍了如何打测试包以及上传相关的分发平台，也提到了脚本自动打包。现在自己负责几个项目的维护与开发工作，发现打一个包出来到上传到iTunes Connect上实在是太慢了，步骤也多，比较的耗时间，也不够自动化。后面自己了解了Jenkins持续构建工具，和fastlane自动打包工具，能够提高打包效率，下面开始介绍如何使用这些工具吧。

# 目录

- ## fastlane

- ## Jenkins

# fastlane

![](https://ws1.sinaimg.cn/large/006tNbRwly1fygs2adp2jj31ub0u0k95.jpg)

The easiest way to build and release mobile apps.fastlane* handles tedious tasks so you don’t have to.

一种快捷的方式去构建和发布手机APP，它可以帮我们处理冗长无味的工作。
最新公司项目打包频率增加，每次打包上传到三方分发平台或testflight上，都要耽搁好长的时间，我就在想有没有一种好的方式，帮我省去打包繁琐的过程，而是通过脚本自动打包上产到相应的平台，于是我就研究了一下fastlane工具，这篇博客主要是做相关记录，方便自己以后查阅，同时也希望能给需要的朋友一些参考帮助。

### 相关地址
- GitHub地址：[fastlane](https://github.com/fastlane/fastlane)
- 官方地址：[fastlane.tools](https://fastlane.tools/)
- 文档地址：[官方文档](https://docs.fastlane.tools/)

### 安装fastlane
- 安装command line tools
```
xcode-select --install
```

- Install fastlane
```
brew cask install fastlane
```

- 切换到工程目录，初始化fastlane
```
cd [工程目录]

fastlane init
```

初始化完成之后，工程目录大概是这个样子：

![](https://ws2.sinaimg.cn/large/006tNbRwly1fygra84uobj30oa0jmacm.jpg)

### 配置相关打包脚本

自己也是最近才熟悉了解fastlane相关命令的使用，我现在配置了打包到上传App Store和上传到Testflight上的脚本。我们可以通过不同的脚本配置，打出不同环境的ipa包，也可以直接通过fastlane配置上传APP相关信息到iTunes Connect上，比如应用截图，APP提交审核时一些信息等，自己暂时还没配置上传信息这些，下面只介绍我用到的吧。

- 上传App Store

  ```ruby
  default_platform(:ios)
  
  platform :ios do
  desc "upload appstore lane"
    lane :release do
      # add actions here: https://docs.fastlane.tools/actions
      #pod资源更新
      cocoapods
  
      #增加build版本号,需要先配置build setting
      #increment_build_number
      #自动增加build号
      increment_build_number_in_plist
  
      #scheme_name
      scheme_name = "xxxx"
      #workspace_name
      workspace_name  = "xxxx.xcworkspace"
      #导出ipa路径
      output_directory = "fastlanebuild_ipa"
      #导出名称 
      output_name = "#{scheme_name}_#{Time.now.strftime('%Y%m%d%H%M%S')}.ipa"
      
      #打包,别名build_app
      gym(
           #代码签名方式,"app-store", "ad-hoc", "package", "enterprise", "development", "developer-id"]
          export_method: "app-store",
          export_xcargs: "-allowProvisioningUpdates",
          scheme: scheme_name,
          workspace: workspace_name,
          #build之前先clean，减小包体积
          clean: true,
          output_directory: output_directory,
          output_name: output_name,
          include_bitcode: false
          )
      #发布到AppStore
      upload_to_app_store(skip_metadata: true,
                          #忽略上传截图
                          skip_screenshots: true,
                          force: true)
    end
  ```

  上传到App Store主要用到了gym，upload_to_app_store命令（他们都有别名，官方文档有描述），他们的配置参数都可以在fastlane官方文档里查到，通过简单的几行的脚本配置，就能打包上传App Store啦，是不是很方便easy！

  - 上传Testflight

    ```ruby
    default_platform(:ios)
    
    platform :ios do
    desc "upload to testflight lane"
      lane :beta do
        #保证推送证书可用
        get_push_certificate
        #pod资源更新
        cocoapods
        #代码签名方式,"app-store", "ad-hoc", "package", "enterprise", "development", "developer-id"]
        #sync_code_signing(type: "appstore")
        #自动增加build号
        increment_build_number_in_plist
        #scheme_name
        scheme_name = "xxxx"
        #workspace_name
        workspace_name  = "xxx.xcworkspace"
        #导出ipa路径
        output_directory = "fastlanebuild_ipa"
        #building app
        build_app(scheme: scheme_name,
                  workspace: workspace_name,
                  include_bitcode: false,
                  clean: true,
                  output_name: "#{scheme_name}_#{Time.now.strftime('%Y%m%d%H%M%S')}.ipa",
                  output_directory: output_directory,
                  export_method: "app-store")
        #上传到Testflight
        upload_to_testflight(skip_waiting_for_build_processing: true)
        #slack(message: "Successfully distributed a new beta build")
      end
    ```

脚本配置和上传App Store差不太多，唯一不同是用到了upload_to_testflight命令。

### 总结

> 自己能使用fastlane正常打出ipa包来，倒腾了一两天，网上介绍fastlane使用资料也是一搜一大堆，但大部分介绍都不是很全，最全的资料还是官方文档，这次的折腾也给了我很多启示，要学会看官方文档资料，现在也只用到了fastlane提供的部分功能，遇到的坑不是很多，等以后有更深入的使用，在更新博客与大家分享。

# Jenkins

![](https://ws2.sinaimg.cn/large/006tNbRwly1fygs1hjn5vj31x00myq9g.jpg)

> Jenkins is a self-contained, open source automation server which can be used to automate all sorts of tasks related to building, testing, and delivering or deploying software.
>
> Jenkins can be installed through native system packages, Docker, or even run standalone by any machine with a Java Runtime Environment (JRE) installed.

怎么说呢，我也不太清楚自己是怎么接触到Jenkins持续集成工具的，当我有最初由持续集成概念的时候应该是我学习cocoapods制作私有共有库的时候，有一个GitHub的徽标是表示项目持续集成状态，叫[Travis CI](https://travis-ci.org/)，貌似是开源项目免费，私有项目需要收费。自己那时候发布的pod库Travis CI没有通过，自己倒腾好久了才让这个徽标变为绿色。好了，不闲聊了，关于Jenkins介绍，我这里也不做介绍了，网上资料也是一搜一大堆，我只介绍集成和使用关键的几个步骤吧！

### 相关地址

- GitHub：[jenkins](https://github.com/jenkinsci/jenkins)
- 官方网址：https://jenkins.io/

### 安装Java环境（MAC）

- 因为Jenkins是基于Java语言开发的，首先要检查自己的电脑是否安装Java。没有安装会报以下错误

![](https://ws3.sinaimg.cn/large/006tNbRwly1fygsr4wnd8j30vc090taq.jpg)

错误已经给出提示了，按照提示安装java8吧：

```
brew cask install homebrew/cask-versions/java8
```

通过命令``java -version``查看是否安装成功。

- 安装成功之后配置一下环境变量：

```
#查看java home 目录
/usr/libexec/java_home
输出是
/Library/Java/JavaVirtualMachines/jdk1.8.0_192.jdk/Contents/Home

#之后编辑bash_profile
vi .bash_profile

#编辑以下内容
export JAVA_HOME=/Library/Java/JavaVirtualMachines/jdk1.8.0_192.jdk/Contents/Home
export PATH=$JAVA_HOME/bin:$PATH  
export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar

:wq
保存即可
```



### 安装Jenkin

- Jenkins安装方式有很多，可以在官方下载安装包安装，也可以通过brew安装，但我个人更推荐通过brew：

```
brew install jenkins
```

安装成功之后，会默认打开网址http://localhost:8080，如果没有打开我们在终端手动启动Jenkins

```
jenkins
```

第一次登录需要用初始化密码，将初始化密码拷贝进去就可以登录了，之后需要安装一些默认的插件，等待安装完成就可以进入主界面啦。

### 项目创建

- 创建项目（Create New Job）

  ![](https://ws3.sinaimg.cn/large/006tNbRwly1fyh1dmhh2nj31xs0pgtdp.jpg)

  ![](https://ws2.sinaimg.cn/large/006tNbRwly1fyh1e6hitnj31q20u046x.jpg)

### 常规设置（general）

- 丢弃旧的构建

![](https://ws1.sinaimg.cn/large/006tNbRwly1fyh1wi0sxfj31qp0u0456.jpg)

- 参数化构建过程

  如果我们项目用的git托管，这时候可能会有多个分支，在构建的时候我们需要选择一个分支。这功能我们需要装一个插件**[Git Parameter](https://wiki.jenkins-ci.org/display/JENKINS/Git+Parameter+Plugin)**，然后做以下配置：

  ![](https://ws4.sinaimg.cn/large/006tNbRwly1fyh27arftqj31dz0u0dkr.jpg)


### 源代码管理（Source Code Management）

- 因为我的项目是用git托管的，这里源代码管理我选的git

![](https://ws1.sinaimg.cn/large/006tNbRwly1fyh2d6gsq6j31dx0u07an.jpg)

- Build Trigger

  这里是指怎么触发构建操作，一搬选择定时构建和轮询构建，看情况而定，这里我暂时都没有勾选，需要注意的是时间书写的格式

  ```
  #每天下午18：00构建一次
  H 18 * * *
  
  #每5分钟构建一次
  H/5 * * * *
  ```

  ![](https://ws4.sinaimg.cn/large/006tNbRwly1fyh2kfsk7dj31b60u0jwq.jpg)

### 构建环境（Build Environment）

暂时没有用到，不做介绍啦。

### 构建（Build）

构建完成之后执行的操作，有如下选项

![](https://ws4.sinaimg.cn/large/006tNbRwly1fyh2nuyi25j31g80hejt9.jpg)

这里我认为是比较重要的地方啦，build成功之后，这里我们可已选择执行shell打包脚本，根据脚本配置导出ipa包，也可以执行fastlane命令上传App Store，也可以上传蒲公英或fir.im三方分发平台。

- shell打包脚本

  用苹果提供的原生打包命令编写shell，这里我没有采用这种方式啦，网上也可以搜到相关的脚本，就不做介绍了。

- 执行fastlane命令上传App Store

  ```
  fastlane release
  ```

  **release**是之前介绍fastlane的时候编写的一个上传App Store的脚本，这里我们也可以上传Testflight。

- 上传蒲公英或fir.im三方分发平台

  这里我做了上传蒲公英平台的操作，蒲公英文档也有介绍相关的命令https://www.pgyer.com/doc/view/jenkins_plugin，我也是通过插件的方式，也可以通过命令。

  ![](https://ws4.sinaimg.cn/large/006tNbRwly1fyh307sh6fj31gi0hg40l.jpg)

  - 参数配置，配置好如下两个参数就可以自动帮我们上传到蒲公英平台啦，是不是很方便

![](https://ws2.sinaimg.cn/large/006tNbRwly1fyh33asqc7j31f80gq425.jpg)

### Post-build Actions

构建完成之后的操作，实用的就是发邮件通知构建结果，暂时不做介绍，有兴趣的朋友可以自行研究。

### 总结

> 总算介绍完成了，从零开始倒腾公司的项目，到支持Jenkins持续集成以及后续的操作，中间过程还是经历了很多的坑，但最后的结果还是令自己满意的，自己从中也学到了许多新知识，还是自己太“懒”吧，不然怎么会不折不休倒腾那么久，哈哈。现在打包还是比之前方便了许多，也省去了许多重复乏味的工作。如果文章中有描述不对的地方，还请大家不吝批评指正，自己也是最近才实践自动打包相关的工具，希望能和大家多多学习交流。

# 问题解决

- 自己设置账号密码无法登陆

  去.jenkins目录下，修改config.xml参数，改为false就好了

  ```
  <useSecurity>false</useSecurity>
  ```

# 参考文章

- [Max OS上Java环境变量配置](https://www.jianshu.com/p/d5bfa5c4d86a)
- [使用 Jenkins 插件上传应用到蒲公英](https://www.pgyer.com/doc/view/jenkins_plugin)
- [jenkins构建时支持git选择分支](https://blog.csdn.net/u012076316/article/details/52056107)
- [iOS 持续交付之 Fastlane](https://juejin.im/post/5a7b10bb6fb9a0636263bfd5)

- [手把手教你利用Jenkins持续集成iOS项目](https://www.jianshu.com/p/41ecb06ae95f)