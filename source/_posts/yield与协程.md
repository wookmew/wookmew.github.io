---
title: yield与协程
date: 2017-06-11 12:23:39
tags: [yield, 协程]
categories: Python
---

> 虽然自己没有用到协程，但在现阶段并发需求越来越高的情况下，还是有必要学习一下的。《fluent python》对这块讲解的不错，拿来学习一下,本章节相当于是读书笔记吧。之前有写过一篇关于[yield](http://jblue.xyz/2017/03/07/%E7%94%9F%E6%88%90%E5%99%A8%E4%B8%8Eyield/)的文章，此次在此基础上继续学习。

<!--more-->
## 协程与生成器的区别

对于Python生成器中的yield来说，yield item这行代码会产出一个值，提供给**next(...)**的调用方; 此外，还会作出让步，暂停执行生成器，让调用方继续工作，直到需要使用另一个值时再 调用next()。调用方会从生成器中拉取值。
从句法上看，协程与生成器类似，都是定义体中包含 yield 关键字的函数。可是，在协程 中，yield 通常出现在表达式的右边(例如，datum = yield)，可以产出值，也可以不产出——如果 yield 关键字后面没有表达式，那么生成器产出 None。协程可能会从调用方接 收数据，不过调用方把数据提供给协程使用的是**.send(datum)**方法，而不是**next(...)**函数。通常，调用方会把值推送给协程。
```python
    In [1]: def demo():
    ...:     print 'start'
    ...:     data = yield
    ...:     print 'data: ', data
    In [2]: d = demo()
    In [3]: next(demo)
    start
    In [4]: demo.send(1)
    data:  1
    Traceback (most recent call last)
    ...
    StopIteration:
```
## 协程

#### 四个状态
协程可以身处四个状态中的一个。当前状态可以使用**inspect.getgeneratorstate(...)** 函数确定，该函数会返回下述字符串中的一个。
1. 'GEN_CREATED'--等待开始执行。
2. 'GEN_RUNNING'--解释器正在执行。
3. 'GEN_SUSPENDED'--在 yield 表达式处暂停。
4. 'GEN_CLOSED'--执行结束。 

因为**send**方法的参数会成为暂停的 yield 表达式的值，所以，仅当协程处于暂停状态时才 能调用 send 方法，例如上面例子中的**d.send(1)**。不过，如果协程还没激活(即，状态是 'GEN_ CREATED')，就直接使用send方法的话，会产生报错。因此，始终要调用**next(d)**激活协程,也可以调用**d.send(None)**，效果一样。
最先调用 next(d) 函数这一步通常称为“预激”(prime)协程(即，让协程向前执行到第一个 yield 表达式，准备好作为活跃的协程使用)。

#### 异常与终止
协程中未处理的异常会向上冒泡，传给 next 函数或 send 方法的调用方(即触发协程的对象)。一般的做法是在生成器对象上调用两个方法：**generator.throw(exc_type[, exc_value[, traceback]])**和**generator.close()**。
```
    class DemoException(Exception):
        def demo_exc_handling():            print('coroutine started')             while True:                try:                    x = yield               except DemoException:                     print('DemoException handled. Continuing...')                else:                    print('coroutine received: {!r}'.format(x))                raise RuntimeError('This line should never run.') 
```
运行结果如下：
```
 >>> exc_coro = demo_exc_handling() >>> next(exc_coro) coroutine started >>> exc_coro.send(11) coroutine received: 11
 
 >>> exc_coro.close() >>> from inspect import getgeneratorstate >>> getgeneratorstate(exc_coro) 'GEN_CLOSED'
 
 >>> exc_coro.throw(DemoException) DemoException handled. Continuing... >>> getgeneratorstate(exc_coro) 'GEN_SUSPENDED'
```

## 扩展 yield from
**yield from**在Python3.3中引入，目前2.7还无法使用。
```python
In [1]: def gen1():
     ...:     for c in 'AB':
     ...:         yield c
     ...:

In [2]: list(gen1())
Out[2]: ['A', 'B']

In [1]: def gen2():
     ...:     yield from 'AB'
     ...:

In [2]: list(gen2())
Out[2]: ['A', 'B']
```


