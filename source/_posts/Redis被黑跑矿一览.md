---
title: Redis被黑跑矿一览
date: 2018-01-20 13:21:22
tags: Redis
categories: Linux
---

> 起因是Sentry上捕捉了一些错误，错误指向redis重的一些缓存数据不存在。当时就很奇怪，自己并未设置过期时间，数据居然就消失了。仔细研究了下，发现Redis里面多出了几个奇怪的数据，一个公钥，一个curl请求。不放心htop看了一下，**/tmp/Circle_MI.png**已经把cpu跑满了(四核跑满了三核)。  

<!--more-->  


## 定位一下 
找了一番，发现了应该就是被人植入跑矿程序了,[问题就是这个了](http://bbs.qcloud.com/thread-30706-1-1.html)。漏洞利用条件主要有三：  
1. Redis服务以root账户运行；
2. Redis无密码或弱密码进行认证；  
3. Redis监听在0.0.0.0公网上；  

执行**netstat -alp |grep redis**看了下，自己之前直接执行**/usr/bin/redis-server**, 没有指定配置文件，redis默认开到公网上了。修改/etc/redis/redis.conf，即:  

```shell
bind 127.0.0.1 # 绑定到本地
port 123456 # 更换端口
requirepass password # 可以选择舍不设置密码
```  

运行：**/usr/bin/redis-server /etc/redis/redis.conf**, it's over.  


## 一些注意点
1. 查看crontab里面的任务，不熟悉的全删掉；
2. 查看用户有没有变多，自己这边redis不是以root起的，所以对方只拿到了当前用户权限；
3. 删除一些可疑的文件；
4. 关闭ssh密码登录，这个之前自己偷懒没关；
5. 重装大法好，以上；


