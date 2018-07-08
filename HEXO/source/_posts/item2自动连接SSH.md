---
title: item2自动连接SSH
tags: [item2,SSH,Shadowsocks,Terminal]
date: 2018-07-08 17:06:49
permalink:
categories: MAC
description: 本文主要介绍如何用item2实现免密登录。
image: https://user-gold-cdn.xitu.io/2018/6/10/163e84603eda8ea0?w=1240&h=484&f=jpeg&s=30907
---
<p class="description"></p>

<!-- more -->

### 目录
> 1、编辑命令脚本
> 2、配置item2

#### 1、编辑命令脚本
- 2.1.1 打开item2终端，创建脚本文件**CentOSAutoLoginSSH**（名字可以自定义）文件，保存在一个你指定的文件夹下：
```
//切换文件夹
cd [你要保存的文件夹下]

//创建文件
touch CentOSAutoLoginSSH
```
- 2.1.2 编辑CentOSAutoLoginSSH
```
vim CentOSAutoLoginSSH
```
2.1.3 配置CentOSAutoLoginSSH
```
#!/usr/bin/expect -f  
#搬瓦工控制面板中的SSH Port
  set port 2121  
#默认用户名root
  set user root  
#主机地址
  set host 172.16.10.71  
#密码
  set password mima123456  
  set timeout -1  
   
  spawn ssh -p$port $user@$host  
  expect "*assword:*"  
  send "$password\r"  
  interact  
  expect eof  

:wq
```
编辑完成之后`:wq`保存配置信息。
#### 2、配置item2
- 2.2.1 item2->Preference->Profile添加配置文件，操作如下图所示：
  ![屏幕快照 2018-05-20 下午6.42.05.png](https://user-gold-cdn.xitu.io/2018/6/10/163e84603e9d33cf?w=1240&h=768&f=png&s=247953)
- 2.2.2 测试免密自动登录，选择顶部菜单**Profile**中的**CentOSAutoLoginSSH**，这时可能会报错，因为**CentOSAutoLoginSSH**没有执行权限，需要执行以下命令：
```
chmod u+x /Users/user/.ssh/CentOSAutoLoginSSH
```
![屏幕快照 2018-05-20 下午6.47.14.png](https://user-gold-cdn.xitu.io/2018/6/10/163e846040081c42?w=496&h=326&f=png&s=46574)

然后测试，就实现了免密自动登录了。
- 2.2.3 然后我们就可以查看shadowsocks文件下的配置文件了
```
cat /etc/shadowsocks/config.json
```
### 参考文章
- [使用iTerm2快捷连接SSH](https://blog.csdn.net/fangxiaoji/article/details/50710220)
- [Mac Item2 SSH免密登录Linux 服务器的两种方式](https://blog.csdn.net/jobschen/article/details/52823980)

