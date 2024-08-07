---
title: Hexo实现静态博客部署
date: 2017-03-04 13:22:58
tags: Hexo
categories: Hexo 
---

> 安装实现均在Mac上,不同的操作系统之间有些许差别。因为喜欢简洁的风格，所以选择了现在这个主题。搭建不算很难，但写博客，贵在坚持，共勉。

<!--more-->

# 基础安装

### Git
安装homebrew，然后通过homebrew安装Git，具体方法请参考homebrew的文档

### NVM
参考了一些资料之后，决定使用NVM进行安装。不过后面发现，有NPM可以更方便的管理，在这边还是先使用NVM了

``` bash
curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.33.0/install.sh | bash
```

### 安装Node.js:

``` bash
nvm install stable
```

### 安装 Hexo：

``` bash
npm install -g hexo-cli
```

### 建立仓库

登录你的Github，新建仓库，名为用户名.github.io固定写法，如username.github.io。在项目的Setting中，把GitHub Pages的Source改为master分支，保存。
注意，此处的username不可以随意新取

## 配置Hexo

官方文档在此

``` bash
cd /Users/xx
mkdir Hexo
hexo init Hexo
cd Hexo
npm install
```

执行如下命令，开启hexo服务器：

``` bash
hexo s --debug  # debug模式
```
浏览器中打开网址[http://localhost:4000](http://localhost:4000)，能看到相应的hexo主题。对_config.yml进行网站相关配置，参考官方指南。vim打开_config.yml,在最下面进行修改:

``` bash
deploy:
    type: git
    repository: https://github.com/username/username.github.io.git
   branch: master
```
PS: 冒号后边都要加一个空格
在hexo文件夹目录下执行生成静态页面命令：

``` bash
hexo generate		或者：hexo g
```

再执行配置命令：

``` bash
hexo deploy			或者：hexo d
```

我在这步出现错误ERROR Deployer not found: git,解决的办法是执行

``` bash
npm install hexo-developer-git --save
```

再次执行hexo generate和hexo deploy命令。

### 安装主题

这里自己选的是NexT,官方文档非常详细见此。

### 文章发布

创建新文章：

``` bash
hexo new "Name"
```

名为Name.md的文件会建在目录/Hexo/source/_posts下,这里推荐用Mweb进行MarkDown的编写。每次部署文章的步骤:
``` bash
hexo clean  
hexo g  
hexo d
```
等待1min左右，文章便会刷新出来。

## 绑定域名

在/Hexo/source目录下新建文件名为：CNAME,将自己的域名如：jblue.xyz写入(此处无www)。然后自然是把之前买的xyz的域名进行解析，关于这个域名自己是在阿里云的域名管理买的，在你购买的域名后边点击：解析 –> 添加解析，域名解析里面有些自带解析可以看情况删除掉。
记录类型：CNAME
主机记录：填写@(我填写了两个解析规则，此处一个@，一个www)
记录值：gjblue.github.io. (最后加.，gjblue改为你自己的用户名)，点击保存即可。

## 后记

诸如评论系统，RSS之类都可以在NexT官方文档找到。评论这块之前用了多说，但是多说关闭了，准备以后尝试下网易的吧。

## 关于备份

吃了次大亏，mac断电重启之后，启动索引似乎出现了问题，多次尝试未果后，无奈只能格式化硬盘重装系统。然后整个Hexo下的配置，文章就都丢失了。研究了一下备份这块，鉴于国内网盘的不靠谱，使用GitHub或者iCloud，都是可行的办法。



