---
title: MongoDB安装踩坑记
date: 2017-06-01 21:50:19
tags: MongoDB  
categories: 数据库
---

> 最近在Ubuntu 16.04上安装MongoDB遇到了不少坑，这里做下整理。一开始是以为直接apt-get就可以安装了，然后找了份资料，但是14.04和16.04安装之间还是有所区别的。网上文档误导甚多，着实蛋疼。

<!--more-->

# 安装
#### 文档
[官方安装文档](https://docs.mongodb.com/manual/tutorial/install-mongodb-on-ubuntu/#install-mongodb-community-edition),有不同版本的安装指南。 [Sf](https://stackoverflow.com/questions/41380805/installing-mongodb-on-ubuntu-16-04)上的一个回答也囊括了整个安装过程。

#### 命令行
如果已经安装了MongoDB，先将其删去。
```shell
sudo apt-get purge mongodb-org*
sudo rm -r /var/log/mongodb
sudo rm -r /var/lib/mongodb
```
16.04安装如下：
```shell
sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 0C49F3730359A14518585931BC711F9BA15703C6
echo "deb [ arch=amd64,arm64 ] http://repo.mongodb.org/apt/ubuntu xenial/mongodb-org/3.4 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-3.4.list
sudo apt-get update
sudo apt-get install -y mongodb-org
```

#### 配置
添加系统环境变量设置，添加如下这一行到/home/xxx/.profile文件。
```shell
export LC_ALL=C
export LESSCHARSET=utf-8
```
第二行是为了能够让git log里面正确显示中文,顺手而为。


#### 问题
**sudo service mongod start**之后，运行mongo启动，发现了连接遭拒，如下：
    
```shell
2017-06-13T09:52:52.237+0800 W NETWORK  [thread1] Failed to connect to 127.0.0.1:27017, in(checking socket for error after poll), reason: Connection refused
2017-06-13T09:52:52.237+0800 E QUERY    [thread1] Error: couldn't connect to server 127.0.0.1:27017, connection attempt failed :
connect@src/mongo/shell/mongo.js:237:13
@(connect):1:6
exception: connect failed
```
在运行mongod看看：
```shell
2017-06-13T10:00:16.110+0800 I STORAGE  [initandlisten] exception in initAndListen: 29 Data directory /data/db not found., terminating
```
不知为何/data/db文件夹并没有建立，从而导致了这个问题。

#### 解决
如果是root用户的话，可以直接运行以下命令。非root用户的话，需要添加一下权限。[这里](https://stackoverflow.com/questions/15229412/unable-to-create-open-lock-file-data-mongod-lock-errno13-permission-denied
)有个比较详细的解释，有兴趣可以翻阅下。
```shell
sudo mkdir -p /data/db

// 非root用户
sudo chown -R `id -u` /data/db
service mongod start
```

