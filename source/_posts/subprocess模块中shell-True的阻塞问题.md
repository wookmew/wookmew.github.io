---
title: subprocess模块中shell=True的阻塞问题
date: 2018-01-03 20:17:03
tags: subprocess
categories: Python
---

> 最近在写一个不断读取日志文件的小脚本，使用subprocess模块时自己遇到了奇怪的阻塞问题。在请教dzdx之后，解决了这个问题。简单来讲，就是设置shell=True这一参数时，<font color="red">Popen()</font> args输入问题。

<!--more-->

  
# 起因 
找到一篇[How can I tail a log file in Python?](https://stackoverflow.com/questions/12523044/how-can-i-tail-a-log-file-in-python?answertab=votes),自己的本机环境是OSX+python2.7,select模块并没有poll函数。然后自己又翻阅了别的一些答案，组合起来写了一个很简单的脚本，即：  

```Python  
popen = subprocess.Popen(['tail',  '-f', logname], stdout=subprocess.PIPE, stderr=subprocess.PIPE, shell=True)
while True:
    line = popen.stdout.readline().strip()
    print line
```  

然后就阻塞住了，没有任何输出。然后自己改了一下代码，即把**shell=True**除去：  
  
```Python
popen = subprocess.Popen(['tail',  '-f', logname], stdout=subprocess.PIPE, stderr=subprocess.PIPE)
while True:
    line = popen.stdout.readline().strip()
    print line
```  

没问题了，但这就尴尬了，为啥我加了**shell=True**就跪了。赶紧问人翻文档。然而在文档，我只注意到了**On POSIX with shell=True, the shell defaults to /bin/sh.**。但问题应该不在使不使用这个shell上。因为自己的log是有内容存在的，所以这个**readline**这一块木有问题。但自己一加**shell=True**就阻塞住了，这就很奇怪了。  

# 解决
这时候dzdx给了一些相关链接，比如[这篇文章](https://medium.com/python-pandemonium/a-trap-of-shell-true-in-the-subprocess-module-6db7fc66cdfd)。我这时候才恍然大悟，想起来文档里的描述：
> On POSIX with shell=True, the shell defaults to /bin/sh. If args is a string, the string specifies the command to execute through the shell. This means that the string must be formatted exactly as it would be when typed at the shell prompt. This includes, for example, quoting or backslash escaping filenames with spaces in them. If args is a sequence, the first item specifies the command string, and any additional items will be treated as additional arguments to the shell itself. That is to say, Popen does the equivalent of: **Popen(['/bin/sh', '-c', args[0], args[1], ...])**   

一开始文档便给出了答案，当使用**shell=True**时，list只有第一个参数被执行，即tail。换个写法：  

```Python
popen = subprocess.Popen('tail -f ' + logname, stdout=subprocess.PIPE, stderr=subprocess.PIPE, shell=True)
while True:
    line = popen.stdout.readline().strip()
    print line
```  

ok，这下子问题解决。

# 总结
英语要学好啊，唉。


