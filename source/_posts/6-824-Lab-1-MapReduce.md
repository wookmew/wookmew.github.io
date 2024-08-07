---
title: '6.824 Lab 1: MapReduce'
date: 2018-06-19 15:22:21
tags: [MIT, MapReduce, 分布式]
categories: Go
---

> [MIT](https://pdos.csail.mit.edu/6.824/schedule.html)的课程，学习的原因有二，一是作为自己继续学习[Go](https://golang.org/doc/)的道路；而是学习更多的知识，**Raft**、**RPC**这些自己之前都未曾有过接触，不能没吃过猪肉，也没见过猪跑。借此机会，学习吧。

<!--more--> 

## 关于MapReduce  
MapReduce是Google提出的一个软件架构，用于大规模数据集（大于1TB）的并行运算。概念“Map（映射）”和“Reduce（归纳）”。课程里提供了Google的[文档](https://pdos.csail.mit.edu/6.824/papers/mapreduce.pdf)，我也找到了一份[中文翻译](http://blog.bizcloudsoft.com/wp-content/uploads/Google-MapReduce%E4%B8%AD%E6%96%87%E7%89%88_1.0.pdf)。  
简单来讲就是分布式系统处理海量数据的一种方式，启动master进程分配任务给N个worker执行。Map和Reduce是两种不同的worker。Map函数读取被分配的输入数据片段，输出中间key/value pair值的集合，Reduce函数收集具有相同中间key值的value值，合并这些value值，形成一个较小的value值的集合。比较经典的例子就是课程当中的wordcount了。  
```  
Input1 -> Map -> a,1 b,1 c,1
Input2 -> Map ->     b,1
Input3 -> Map -> a,1     c,1
                    |   |   |
                    |   |   -> Reduce -> c,2
                    |   -----> Reduce -> b,2
                    ---------> Reduce -> a,2  
```  


## Part 1  Map/Reduce input and output  
part1的作业内容可以结合课程与代码中的注释，还是挺好理解的。






