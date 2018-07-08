---
title: GitPages+Hexo搭建个人博客
tags: [Hexo,GitHub,GitPages]
date: 2018-07-07 23:24:44
permalink:
categories: Hexo
description: 这篇文章主要介绍如何用GitPages+Hexo搭建个人博客，如果你也有兴趣，那么这篇文章对你将有所帮助。
image: https://ws2.sinaimg.cn/large/006tNc79ly1ft1psxc65cj31kw0m5tcg.jpg
---
<p class="description"></p>

<!-- more -->

如果你想从零开始搭建一个属于自己的静态博客网站，可以参考下面三篇博文，这三篇博文是记录我从零开始搭建自己静态博客的全过程，希望能给需要的朋友一些参考：

- [GitPages+Hexo搭建个人博客](https://www.jianshu.com/p/201283bcd64a)
- [Hexo相关配置和使用](https://www.jianshu.com/p/d5d3e10576d1)
- [Hexo-NexT配置超炫网页效果](https://www.jianshu.com/p/9f0e90cc32c2)

最终成果： [blogwenbo.com](http://blogwenbo.com/)

### 一、GitHub创建项目
- 1.1 使用GitPages搭建自己静态博客前提要注册申请GitHub账号。GitHub相关配置可参考这篇文章：[MAC上Git安装与GitHub基本使用](https://www.jianshu.com/p/7edb6b838a2e)。
- 1.2 GitHub上新创建一个[wenmobo.github.io](https://github.com/wenmobo/wenmobo.github.io)仓库，**wenmobo**是我的账号名，这里替换成你自己的就可以了。项目格式名称为`[用户名].github.io`，如下：
```
username.github.io
```
创建成功之后如下：
![屏幕快照 2018-01-14 下午11.52.25.png](http://upload-images.jianshu.io/upload_images/3072214-224c6bb8fcdf8ffd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
- 1.3 在桌面新建一个文件夹**MyBlog**，`cd`到该文件夹，将项目克隆到本地：
```
git clone git@github.com:wenmobo/wenmobo.github.io.git
```
### 二、安装[Node.js](https://nodejs.org/en/)
- 2.1 Node.js支持用HomeBrew安装，首先要安装Homebrew，Homebrew安装可查看这篇文章：[MAC上Homebrew常用命令整理](https://www.jianshu.com/p/c60789934af1)
- 2.2 Homebrew安装好之后，用Homebrew安装Node.js，终端输入：
```
brew node
```
### 三、安装[hexo](https://hexo.io/zh-cn/)
- 3.1 安装hexo，终端输入：
```
npm install -g hexo-cli
```
- 3.2 在本地仓库**MyBlog**新建文件夹**Blog**，然后在终端`cd [Blog文件夹路径]`，执行以下命令初始化博客：
```
hexo init

//或者
hexo init <folder>
cd <folder>
npm instal
```
成功之后，目录文件如下：
```
.
├── _config.yml
├── package.json
├── scaffolds
├── source
|   ├── _drafts
|   └── _posts
└── themes
```
- 3.3 查看本地效果，终端输入：
```
hexo s
```
终端输出：
```
WMBdeMacBook-Pro:Hexo WENBO$ hexo s
INFO  Start processing
INFO  Hexo is running at http://localhost:4000/. Press Ctrl+C to stop.
```
将http://localhost:4000/.拷贝到Chome浏览器，查看效果，如下图所示：
![屏幕快照 2018-01-14 下午11.01.57.png](http://upload-images.jianshu.io/upload_images/3072214-9ce7b0f78ef70404.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
### 四、总结
> GitHubPages+Hexo搭建静态博客的准备工作到这里都完成了，下面一篇文章：[Hexo相关配置和使用](https://www.jianshu.com/p/d5d3e10576d1)会介绍Hexo相关配置。
### 五、参考文章
1、[GithubPages教程 在GithubPages上搭建个人主页](http://blog.csdn.net/yanzhenjie1003/article/details/51703370)
2、[在Github上使用Hexo搭建个人博客](https://itgoyo.github.io/2019/12/28/%E5%9C%A8Github%E4%B8%8A%E4%BD%BF%E7%94%A8Hexo%E6%90%AD%E5%BB%BA%E4%B8%AA%E4%BA%BA%E5%8D%9A%E5%AE%A2.html)
3、[如何使用hexo搭建个人博客（Mac OS系统，windows仅作参考）](https://www.jianshu.com/p/6d2ec4ca6186)
4、[Hexo博客主题推荐](https://www.jianshu.com/p/bcdbe7347c8d)