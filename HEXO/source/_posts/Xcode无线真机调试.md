---
title: Xcode无线真机调试
tags: [Xcode,iOS,iPhone,LLDB]
date: 2018-07-08 16:58:29
permalink:
categories: iOS
description: 看到安卓的小伙伴都能用无线调试APP，好生羡慕，为什么Xcode不支持无线调试呢，索性自己也在Google上搜一搜发现Xcode9是支持无线调试的，虽然自己平时也不怎用，还是做下记录吧。
image:
---
<p class="description"></p>

<!-- more -->

###  一、硬性条件
- 硬件环境
  MAC、Xcode9
- 系统
  Mac OSX 10.12.5、iOS11
### 二、具体操作步骤
- 将手机用数据线连接到MAC，Xcode->Devices And Simulators->Devices中勾选**connect via network**
  ![屏幕快照 2018-06-05 下午10.40.47.png](https://user-gold-cdn.xitu.io/2018/6/10/163e852cb5e36b84?w=1240&h=871&f=png&s=181947)
- 点击手机图标，鼠标右键，配置局域网Connect via IP Address
  ![屏幕快照 2018-06-05 下午10.44.35.png](https://user-gold-cdn.xitu.io/2018/6/10/163e852cba4f9aa5?w=1240&h=865&f=png&s=147034)
- 配置完成之后手机图标会有一个地球标志，带表已经连接成功
- 最后运行项目，就可以无线调试啦