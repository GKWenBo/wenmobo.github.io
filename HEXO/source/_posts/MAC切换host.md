---
title: MAC切换host
tags: [MAC,iOS]
date: 2019-06-03 21:19:12
permalink:
categories: MAC
description: 在日常开发中，有时我们需要切换不同的开发环境测试，切换多个hosts，下面介绍MAC电脑如何切换hosts。
image:
---
<p class="description"></p>



在日常开发中，有时我们需要切换不同的开发环境测试，切换多个hosts，下面介绍MAC电脑如何切换hosts。

<!-- more -->

### 前言

在最初接触到iOS开发，切换正式与测试网络环境是通过切换网络地址域名的方式，本地定义一个宏定义实现域名的切换，这种方式确实比较的方便。后来自己又了解了另外一种切换网络环境的方式，就是切换host，在工程代码里面不需要做额外的配置。最开始一直使用同事配置好的环境，但后来发现有时还是不太方便，于是在自己的电脑也倒腾了一下，最后也能成功在真机上切换到正式测试环境，期间也遇到许多坑，下面介绍一下如何配置。

### SwitchHosts介绍与安装

#### [官方网址](https://oldj.github.io/SwitchHosts/#cn)

#### [GitHub](https://github.com/oldj/SwitchHosts/)

#### 安装

- 通过终端命令安装

  ```
  brew cask install switchhosts
  ```

- 安装成功之后

  ![](https://ww3.sinaimg.cn/large/006tNc79ly1g3obhfr0klj318i0rk41r.jpg)

- 将需要添加的host粘贴到指定的host名下即可，打开开关即可切换(**注意：这里电脑的host是可以切换成功，当你用真机连上电脑共享出的wifi，发现还是没有切换，这时需要用到Dnsmasq工具**)

### Dnsmasq安装与配置

> DNSmasq是一个小巧且方便地用于配置[DNS](https://baike.baidu.com/item/DNS/427444)和[DHCP](https://baike.baidu.com/item/DHCP)的工具，适用于小型[网络](https://baike.baidu.com/item/网络/143243)，它提供了DNS功能和可选择的DHCP功能。它服务那些只在本地适用的域名，这些域名是不会在全球的DNS服务器中出现的。DHCP服务器和DNS服务器结合，并且允许DHCP分配的地址能在DNS中正常解析，而这些DHCP分配的地址和相关命令可以配置到每台[主机](https://baike.baidu.com/item/主机/455151)中，也可以配置到一台核心设备中（比如路由器），DNSmasq支持静态和动态两种DHCP配置方式。
>
> ​																																						—百度百科

#### 安装

```
brew link dnsmasq
```

#### 配置文件

```
resolv-file=/usr/local/etc/resolv.dnsmasq.conf
strict-order
listen-address=127.0.0.1
addn-hosts=/usr/local/etc/dnsmasq.hosts
conf-dir=/usr/local/etc/dnsmasq.d
cache-size=10000
```

名词解释：

- `resolv-file`     上游DNS服务配置
-  `strict-order`    严格按照上述文件中的配置顺序执行
- `listen-address` 监听请求的地址（127.0.0.1：仅本机，0.0.0.0：任何人）
-  `addn-hosts`      一些你需要的解析结果
-  `conf-dir`        其他配置路径
-  `cache-size`      缓存大小

#### 使用

```
//停止服务
sudo brew services stop dnsmasq
//重启服务
sudo brew services restart dnsmasq
//刷新DNS缓存
sudo killall -HUP mDNSResponder
```

**注意：点击SwitchHosts切换后，如果发现未切换到想要的环境可按顺序执行以上命令**

#### 手机配置

删除默认DNS，添加电脑的IP地址，如下图所示：

![](https://ww4.sinaimg.cn/large/006tNc79ly1g3oboijx8cj30u01hc0ve.jpg)

最后打开手机访问发现已经连上测试环境了，超开心吧O(∩_∩)O~~。