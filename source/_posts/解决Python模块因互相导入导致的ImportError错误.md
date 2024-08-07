---
title: 解决Python模块因互相导入导致的ImportError错误
date: 2017-03-20 13:02:55
tags: import
categories: Python
---

> 今天在编写执行测试用例时，报了ImportError错误。很是困惑，因为自己引用相当于是同级目录互调，折腾一下午才发现在写测试用例的时候import顺序正好在项目代码中形成了循环引用。

## 相互导入示例
<!--more-->
``` python
# test1.py
from test2 import b
def a():
    b()
# test2.py
from test1 import a
def b():
    a()
Traceback (most recent call last):
  File "test.py", line 4, in <module>
    import test1
  File "/Users/jblue/Code/exe/test1.py", line 3, in <module>
    from test2 import b
  File "/Users/jblue/Code/exe/test2.py", line 3, in <module>
    from test1 import a
ImportError: cannot import name a
```

## 调试方法
自己也是第一次遇到这种问题，因为出现ImportError问题的情况还是很多的。比如路径问题，重名等等。主要使用PyCharm打断点进行调试，在控制台也可以进行调试，比如sys.modules.

## 解决
相互导入解决的根本办法还是设计功能模块时就应该规避这一点，要有良好的层次结构。
如果是接手项目之类，可以调整导入的顺序，或者局部导入。

``` python
# test1.py
def a():
    from test2 import b
    b()
```


