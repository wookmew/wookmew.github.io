---
title: Flask + Gunicorn + Nginx 部署
date: 2017-03-04 18:02:38
tags: Flask
categories: Python
---
> 翻阅了网上的一些资料之后，还是决定使用Flask＋Gunicorn＋Nginx 部署Django之类框架也是类似，不过功利地讲，还是Django招聘职位更多一些。

<!--more-->

## 初始准备

代码托管于阿里云，选择的是Centos7，一来是为了尝鲜，二来Centos7使用的是Python2.7。如果选择6.5版本的话，yum需要处理下(原由Python2.6实现，需替换)。Flask,Nginx,Mysql等环境安装，网上教程很多，在此不做复述。

## Gunicorn

Gunicorn是一个Python WSGI UNIX的HTTP服务器。它既支持 eventlet，也支持greenlet。
``` bash
sudo pip install gunicorn  # 安装  
gunicorn myproject:app  # 启动
```

Gunicorn提供了许多命令行选项，见gunicorn -h。例如，用二个worker 进程（gunicorn -h）来运行一个Flask应用，绑定到localhost的4000端口:

gunicorn -w 2 -b 127.0.0.1:4000 manage:manager  # 进程数需考虑核数

## Nginx

一个简单的nginx配置，它监听localhost的4000端口。
``` bash
  server {
    listen 80;
    server_name www.yoururl.com;

    access_log  /var/log/nginx/access.log;
    error_log  /var/log/nginx/error.log;

    location / {
        proxy_pass         http://127.0.0.1:4000/;
        proxy_redirect     off;

        proxy_set_header   Host             $host;
        proxy_set_header   X-Forwarded-For  $proxy_add_x_forwarded_for;
    }
}
```

## 一些后续

之前自己买了.xyz域名，兴致勃勃地准备去备案，却发现.xyz在京无法备案。鉴于.com自己想要的域名太贵，还是去搭个Hexo的静态博客吧。

