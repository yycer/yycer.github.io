---
layout: post
title: 并发 - [10] Java内存模型
category: multithread
tags: [multithread]
excerpt: Concurreny Java Memory Model  
---

以下内容总结自：   
  
`徐隆曦 - Java 并发编程 78 讲 - [11]Java 内存模型`   


## Java Memory Model  

首先，`JMM`有什么用？  

在更早期的语言中，其实并不存在`内存模型`的概念。  

所以`程序`最终执行的效果往往依赖于具体的`处理器`，而不同的处理器的规则又不一样，  

因此同样一段代码，可能在`处理器A`上运行正常，而在`处理器B`上运行的结果却不一致。  

同理，在没有`JMM`之前，不同的`JVM`的实现，也会带来不一样的`"翻译"`结果。  

所以`Java`非常需要一个标准，来让`Java开发者`、`编译器工程师`和`JVM工程师能够达成一致`。  

达成一致后，我们就可以很清楚的知道什么样的代码最终可以达到什么样的运行结果，  

让`多线程`运行结果可以预期，这个标准就是`JMM`，这也就是需要`JMM`的原因。  


然后，明确`JMM`最重要的三点内容：  

- `重排序`  
- `原子性`  
- `内存可见性`  


重排序的目的是什么？  

> 减少相关指令执行次数，从而提高整体的运行效率。  


再来介绍一下`JMM`中的`主内存`和`工作内存`：  


来看下示意图：  

该图来源于 `徐隆曦 - Java 并发编程 78 讲`

`[第60讲：主内存和工作内存的关系？]`  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/work_main_memory.png)  


`JMM`对于`主内存`和`工作内存`有如下规定：  

第一，所有的变量都存储在`主内存`中，而每个线程都拥有`独立的工作内存`，其中的变量是主内存的拷贝。  

第二，线程不能直接读/写主内存中的变量，仅可以操作自己工作内存中的副本。  

第三，主内存是由多个线程所共享的，但线程间不能共享各自的工作内存，  
如果线程间需要通信，则必须借助主内存中转完成。



## Happens-Before原则  


什么是`Happens-Before`原则？  

它用于描述和`可见性`相关的问题，如果第一个操作`Happens-Before`第二个操作，  

那么第一个操作对于第二个操作一定是可见的，  

也就是说，第二个操作执行时，就一定能看到第一个操作的执行结果。  

来看三点最重要的规则：  

如果分别有`操作x`和`操作y`，用`hb(x, y)`表示`x Happens-Before y`  

### 单线程规则  

在一个独立的线程中，按照程序代码的执行流顺序执行操作，  

如果`操作x`在`操作y`之前，那么`hb(x, y)`  

### 锁操作规则  

如：`synchronized`和`Lock`接口等。  

如果`操作x`是`解锁`，而`操作y`是对同一个锁的`加锁`，那么`hb(x, y)`   

### volatile变量规则  

对于一个`volatile`变量的写操作`Happens-Before`后面对于该变量的读操作，  

这就代表了，如果某个变量被`volatile`修饰，那么每次修改之后，  

其他线程再去读取这个变量时，就一定能读取它的最新的值。  
