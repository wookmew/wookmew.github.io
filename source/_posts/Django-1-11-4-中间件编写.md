---
title: Django 1.11.4 中间件编写
date: 2018-03-25 12:23:38
tags: Django
categories: Python
---

> 目前服务器所用Django版本是1.11.4,想要对Django进行简单的性能分析。一开始是准备写个简单的装饰器利用time.time()进行统计，但这样实在是有些麻烦，然后准备插个中间件来写。  

<!--more-->  

##  问题
 之前没写过，网上一搜开始写，但是问题就来了。比如[这篇](http://www.cnblogs.com/OldJack/p/7112811.html)，但不管怎么尝试都不对。错误还是挺多，几番调试下来缺少**get_response**方法。感觉到有些不对，发现也有写法是调用**MiddlewareMixin**进行继承，简单看下MiddlewareMixin源码发现了不少问题。  
 
```Python
 class MiddlewareMixin(object):
    def __init__(self, get_response=None):
        self.get_response = get_response
        super(MiddlewareMixin, self).__init__()

    def __call__(self, request):
        response = None
        if hasattr(self, 'process_request'):
            response = self.process_request(request)
        if not response:
            response = self.get_response(request)
        if hasattr(self, 'process_response'):
            response = self.process_response(request, response)
        return response
```   

 自己手头上看的1.8的文档，并没有关于这些的说明。感觉问题和**response = self.get_response(request)**是跑不掉了，这时候突然想到自己版本用的是1.11.4，跑到官网一看，果然是[变了](https://docs.djangoproject.com/en/1.11/topics/http/middleware/)。但是对**get_response**还是感觉到奇怪，文档是这么描述：**Middleware factories must accept a get_response argument. You can also initialize some global state for the middleware. Keep in mind a couple of caveat**。  
 翻了翻官方是怎么写的，比如[cache](https://github.com/django/django/blob/1.11.4/django/middleware/cache.py)这个中间件。  
 
```Python
 def __init__(self, get_response=None):
        self.cache_timeout = settings.CACHE_MIDDLEWARE_SECONDS
        self.key_prefix = settings.CACHE_MIDDLEWARE_KEY_PREFIX
        self.cache_alias = settings.CACHE_MIDDLEWARE_ALIAS
        self.cache = caches[self.cache_alias]
        self.get_response = get_response
```   
 
照这个思路，写了个很简易的中间件：  
 
```Python
 # -*- coding: utf-8 -*-
import pstats
import cStringIO as StringIO
from django.conf import settings
from django.utils.deprecation import MiddlewareMixin

try:
    import cProfile as profile
except ImportError:
    import profile


class ProfilerMiddleware(MiddlewareMixin):

    def __init__(self, get_response=None):
        self.profiler = profile.Profile()
        self.get_response = get_response

    def can(self, request):
        if settings.DEBUG:
            return True

    def process_request(self, request):
        if self.can(request):
            self.profiler.enable()
        return None

    def process_response(self, request, response):
        if self.can(request):
            self.profiler.disable()

            io = StringIO.StringIO()

            ps = pstats.Stats(self.profiler, stream=io)
            ps.strip_dirs().sort_stats('tottime')
            ps.print_stats(10)
            print io.getvalue()

        return response
```  
 
 

##  一些后续
 后来了解到也可以写wsgi middleware进行操作，比如这篇[文章](http://www.giantflyingsaucer.com/blog/?p=4877)介绍的这样，但暂时还是按Django的来吧




