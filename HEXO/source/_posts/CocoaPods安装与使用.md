---
title: CocoaPods安装与使用
tags: [CocoaPods,MAC,iOS]
date: 2018-08-01 21:30:38
permalink:
categories: iOS
description: 在开发项目的时候，难免会导入一些三方开源库，CocoaPods是OS X管理三方开源库的工具，用这个工具，我们可以轻松集中管理、更新三方开源库。下面开始介绍CocoaPods安装与使用吧
image:
---
<p class="description"></p>

<!-- more -->

### 更新日志

2018-08-01：整理文章目录结构，添加忽略CocoaPods警告方法，解决出现OTHER_LDFLAGS方法。

**介绍内容目录**

- 一、安装RVM
- 二、配置RubyGems
- 三 、CocoaPods安装
- 四 、CocoaPods使用
- 五、问题解决
### 一、安装[RVM](https://rvm.io/)
- 安装RVM命令如下：
```
curl -L get.rvm.io | bash -s stable 
```
- 查看rvm版本
```
rvm -v
```
- 更新RVM
```
rvm get stable
```
- 查看可下载的ruby版本
```
rvm list known
```
输出结果为：
```
# MRI Rubies
[ruby-]1.8.6[-p420]
[ruby-]1.8.7[-head] # security released on head
[ruby-]1.9.1[-p431]
[ruby-]1.9.2[-p330]
[ruby-]1.9.3[-p551]
[ruby-]2.0.0[-p648]
[ruby-]2.1[.10]
[ruby-]2.2[.7]
[ruby-]2.3[.4]
[ruby-]2.4[.1]
ruby-head
```
- 选择版本安装
```
//安装2.4.1
rvm install 2.4.1
```
安装如果遇到如下错误：
![屏幕快照 2017-12-17 下午1.59.45.png](https://user-gold-cdn.xitu.io/2018/1/24/161288dc7e81a81c?w=1138&h=120&f=png&s=30306)
安装Command Line Tools即可
```
xcode-select --install
```
- 查看已安装的版本
```
rvm list
```
输出结果如下：
```
rvm rubies
=* ruby-2.4.1 [ x86_64 ]
# => - current
# =* - current && default
#  * - default
```
- 查看当前使用的版本
```
rvm current
```
- 设置默认版本
```
rvm use 2.4.1 --default
```
- 删除安装过的版本
```
rvm remove 2.2.2
```
如果提示权限不足，同理加上sudo
```
sudo rvm remove 2.2.2
```
### 二、升级[RubyGems](https://rubygems.org/?locale=zh-CN)
- 在终端输入：
```
gem update --system
```
若果是最新，则输出：
```
Latest version currently installed. Aborting.
```
- 若果没有权限报错，在命令前加上sudo
```
sudo gem update --system
```
- 更换源（最新使用的是：https://gems.ruby-china.org/）
```
gem sources --remove https://rubygems.org/

gem sources -a https://gems.ruby-china.org/
```
-  查看ruby镜像
```
gem source -l
```
输出结果：
```
https://gems.ruby-china.org/
```
### 三、[CocoaPods](https://guides.cocoapods.org/using/getting-started.html#getting-started)安装
- 终端输入
```
sudo gem install cocoapods
或
sudo gem install -n /usr/local/bin cocoapods
```
- 安装了多个xcode进行选择
```
sudo xcode-select -switch /Applications/Xcode.app/Contents/Developer
```
- 安装本地库
```
pod setup
```
执行上面的命令过后，会出现卡主不动，这个是时候是在下载，通常会等很久。这是后可以通过`cmmand+n`新创一个终端窗口，然后`cd ~/.cocoapods/ `到该文件下，执行`du -sh *`查看大小：
```
1015M	repos
```
- 查看版本
```
pod --version
```
- 升级CocoaPods
```
sudo gem install -n /usr/local/bin cocoapods
或
sudo gem install cocoapods
```
### 四、CocoaPods使用
#### 工程导入三方库
-  创建一个工程test，终端切换到工程路径：
```
cd 工程路径
```
- 终端输入：
```
pod init
```
这时工程就会生成一个podfile
![屏幕快照 2017-12-17 下午5.39.24.png](https://user-gold-cdn.xitu.io/2018/1/24/161288dc7ef162b2?w=1240&h=136&f=png&s=49290)

- 编辑podfile：
```
vim podfile
```
进入之后按`i`进入编辑模式，添加三方开源库如：`pod 'AFNetworking'`(也可指定版本`pod 'AFNetworking', '~> 3.1.0'`)，然后输入`:wq`回车保存。
```
# Uncomment the next line to define a global platform for your project
# platform :ios, '9.0'
pod 'AFNetworking'
target 'test' do
  # Uncomment the next line if you're using Swift or would like to use dynamic frameworks
  # use_frameworks!

  # Pods for test

end
~                                                                               
~                                                                               
~                                                                                                                                                        
:wq
```
- 执行安装
```
pod install
或
pod install --no-repo-update
```
好了，到此CocoaPods的安装与使用都介绍完毕。
![屏幕快照 2017-12-17 下午5.47.41.png](https://user-gold-cdn.xitu.io/2018/1/24/161288dc7f1688b0?w=402&h=264&f=png&s=26984)
#### 更新三方库
- 更新所有三方库
```
//更新所有可更新的三方库
pod update
```
- 间接更新三方库
```
cd ~/.cocoapods
ls
cd repos
ls
cd master
ls
ls -a
git pull
```
- 更新指定库
```
pod update Masonry
```

- 省时更新方法
```
pod update --verbose --no-repo-update
```
#### 移除导入过的三方库
- 第一步：编辑**podfile**，将需要删除的三方库移除掉
  ~~pod 'AFNetworking', '~>3.1.0'~~
- 第二步：执行`pod install`，重新编译，如果没有报错则移除成功
```
pod install
```
#### 移除工程中CocoaPods
- 删除工程文件夹下的Podfile、Podfile.lock和Pods文件夹
- 删除xcworkspace文件
- 打开xcodeproj文件，删除项目中的libpods.a和Pods.xcconfig引用
- 打开Build Phases选项，删除Check Pods Manifest.lock和Copy Pods Resources
  主要就是上面四个步骤。
### 五、问题解决
#### 1、执行`gem update --system`报证书错误，在网上找了很久也没有找到解决方法，后来还是找到了,方法是忽略证书验证。
步骤：
前往`~/.gemrc`，打开文件，并添加`:ssl_verify_mode: 0`
```
---
:backtrace: false
:bulk_threshold: 1000
:sources:
- https://gems.ruby-china.org/
:update_sources: true
:verbose: true
:ssl_verify_mode: 0
```
`cmmand+s`保存，然后在执行`sudo gem update --system`，更新成功。
#### 2、[Unable to require openssl, install OpenSSL and rebuild ruby](https://stackoverflow.com/questions/21201493/couldnt-require-openssl-in-ruby)
```
//如果没有安装openssl，则用honebrew安装
brew install openssl

//重装rvm并关联openssl
rvm reinstall 2.4.0 --with-openssl-dir=`brew --prefix openssl`
```
如果安装了**2.4.0**版本则重新安装，没有安装则安装，安装成功之后，就能`sudo gem update --system`正常更新了。

#### 3、CocoaPods 出现 OTHER_LDFLAGS 错误，如下图所示

![](https://ws4.sinaimg.cn/large/006tKfTcly1ftuj9ttidrj30vk0e8q5o.jpg)

- **解决方法1**：**Target**-->**Build Settings**-->**Other Linker Flags**中添加`$(inherited)`，之后执行`pod install`或`pod update`警告就会消失。
- **解决方法2**:`project.xcodeproj`右键显示包内容，用文本编辑器打开 `project.pbxproj`，`command + F` 搜索 `OTHER_LDFLAGS` ，删除搜索到的设置，`command + S` 保存，然后重新执行 `pod install` 或者 `pod update` 。

#### **4、Cocoapods第三方库编译提示warning的解决方法**

- 忽略所有警告

  ```
  inhibit_all_warnings!
  ```

  之后执行`pod install` 或 `pod update` 即可。

- 忽略单个库警告

  ```
  pod 'Masonry', :inhibit_warnings => true
  ```

  之后执行`pod install` 或 `pod update` 即可。

#### 总结

> CocoaPods安装与使用就介绍到这里了，如果在以后CocoaPods安装使用工程中遇到问题，如果找到了解决方案，我也会贴出来。
#### 参考文章
- [iOS 删除已经配置的类库和移除CocoaPods](https://www.jianshu.com/p/552f21a989ba)
- [使用CocoaPods（二）删除已经配置的类库和移除CocoaPods](http://blog.csdn.net/jymn_chen/article/details/19213601)
- [【iOS 开发】解决使用 CocoaPods 执行 pod install 时出现 - Use the `$（inherited）` flag ... 警告](https://www.jianshu.com/p/dfb2a5834cd0)
- [Cocoapods第三方库编译提示warning的解决方法](http://zengyi.me/blog/2015/01/09/cocoapodsdi-san-fang-ku-bian-yi-ti-shi-warningde-jie-jue-fang-fa/)