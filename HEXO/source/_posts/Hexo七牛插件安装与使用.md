---
title: Hexo七牛插件安装与使用
tags: [Hexo,QiNiu]
date: 2018-07-08 18:37:19
permalink:
categories: Hexo
description: 本篇博客主要讲解七牛云插件安装与使用，博客中获取七牛云存储的图片主要有两种方式，一种是在七牛控制台中上传图片然后取图片外链地址，另外一种是用七牛云插件+标签语法将图片上传到七牛云并显示到我们所写的博客中，这里主要介绍第二种方式。
image:
---
<p class="description"></p>

<!--more-->

### **相关网址**

- [hexo-qiniu-sync](https://github.com/gyk001/hexo-qiniu-sync)
- [七牛云](https://www.qiniu.com/)
### 注册七牛个人账号
- 首先需要到[七牛云](https://www.qiniu.com/)官网注册个人账号。
- 进入控制台创建存储空间，如下图所示：
  ![创建存储空间.png](https://user-gold-cdn.xitu.io/2018/1/24/16128a78182fc141?w=1240&h=630&f=png&s=161064)
- 将融合 CDN默认域名添加到[万网](https://www.aliyun.com/?spm=a2c1d.8251892.0.0.7b61a3c0xNylGH)中，配置格式如下图所示：
  ![添加到万网.png](https://user-gold-cdn.xitu.io/2018/1/24/16128a781811be5d?w=1240&h=78&f=png&s=22185)
### 安装七牛云插件
- 安装七牛云插件
  ```
  npm install hexo-qiniu-sync --save
  ```
### 配置相关信息
- 配置站点文件**_config.yml**，配置入内容（**注意**：添加到配置文件时，把//去掉）
```
#plugins:
# - hexo-qiniu-sync

qiniu:
  offline: false
  sync: true
  bucket: blogwenbo
  //这里将其注释掉，不注释，执行hexo g报错
  # secret_file: sec/qn.json or C:
  access_key: your access_key
  secret_key: your secret_key
  // 上传的资源子目录前缀.如设置，需与urlPrefix同步
  dirPrefix: static
  //外链前缀
  urlPrefix: http://p2zukkwm9.bkt.clouddn.com/static
  //使用默认配置即可
  up_host: http://upload.qiniu.com
  //本地目录
  local_dir: static
  // 是否更新已经上传过的文件(仅文件大小不同或在上次上传后进行更新的才会重新上传)
  update_exist: true
  image: 
    folder: images
    extend: 
  js:
    folder: js
  css:
    folder: css
```
- 生成七牛配置路径，执行下面命令任意一个
  ```
  hexo s
  或
  hexo g
  //终端输出
  INFO  -----------------------------------------------------------
  INFO  qiniu state: online
  INFO  qiniu sync:  true
  INFO  qiniu local dir:  static
  INFO  qiniu url:   http://blogwenbo.com/static
  INFO  -----------------------------------------------------------
  INFO  Start processing
  INFO  Now start qiniu sync.
  INFO  Need upload file num: 0
  ```
  就会在**static**目录下生成**images**、**css**、**js**三个文件夹。这时我们把测试图片**七牛云.png**放在**images**文件夹下，然后按照如下标签语法书写：
  ```
  {% qnimg 七牛云.png title:七牛云 alt:七牛云 'class:' extend:?imageView2/2/w/450 %}
  ```
  ![七牛云.png](https://user-gold-cdn.xitu.io/2018/1/24/16128a78184a6943?w=300&h=300&f=png&s=1759)

  - 同步静态资源到七牛云空间，主要有两种方式，一种是使用hexo命令，还有一种是使用七牛插件命令，可以参考GitHub文档：[hexo-qiniu-sync](https://github.com/gyk001/hexo-qiniu-sync)
  ```
  //1、启用本地服务器.即使用 hexo server 命令（简写为 hexo s）
  //当以本地服务器模式启动后，会自动监测 local_dir 目录下的文件变化， 会自动将新文件进行上传。
  如果文件进行了修改，但设置中没有启用 update_exist 配置，则不会更新到七牛空间。
  hexo s
  hexo server
  
  //2、使用命令行命令(sync | s | sync2 | s2)
  //命令行命令会扫描 local_dir 目录下的文件，同步至七牛空间。
  hexo qiniu sync
  hexo qiniu s
  hexo qiniu sync2
  hexo qiniu s2
  ```
  - 图片处理可参考官方文档
    [图片基本处理](https://developer.qiniu.com/dora/manual/1279/basic-processing-images-imageview2)
### 问题解决
- 没有注释**secret_file: sec/qn.json or C:**报错，如下图所示：
  ![HexoError.png](https://user-gold-cdn.xitu.io/2018/1/24/16128a7818576183?w=1142&h=250&f=png&s=48638)

  ```
    # secret_file: sec/qn.json or C:
  ```
- [hexo-qiniu-sync安装好后，hexo s命令不见了，hexo d也提示问题 #41](https://github.com/gyk001/hexo-qiniu-sync/issues/41#issuecomment-279229378)
  ```
  //将其注释就好了
  #plugins:
  # - hexo-qiniu-sync
  ```
### 结语
自己也参考了一些文章，但讲解的都不是很详细完整，有些博客只讲解了重要的一些步骤，不管怎样，最后自己还是捣腾出来了，还是挺折腾人的，我也是第一次用七牛云存储图片，有些地方可能讲解的不是很完整，也请谅解。希望本篇博客能给你一些帮助，也欢迎大家一起交流学习。成功案例：[Hexo七牛插件安装与使用](http://blogwenbo.com/2018/01/23/Hexo%E4%B8%83%E7%89%9B%E6%8F%92%E4%BB%B6%E5%AE%89%E8%A3%85%E4%B8%8E%E4%BD%BF%E7%94%A8/)。
### 参考文章
[Hexo 七牛同步插件的使用](http://www.ixirong.com/2016/10/31/how-to-use-hexo-qiniu-sync-plugin/)