---
layout: post
title: JVM - 内存区域 & 垃圾回收算法
category: jvm
tags: [jvm]
excerpt: JVM 内存区域 & 垃圾回收算法
---

## 内存区域  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/JVM_internal_area.png)  


- 程序计数器`[PC]`： 记录当前线程所执行的字节码的地址。  
- 虚拟机栈`[VM Stack]`： 保存执行方法时所创建的栈帧`[Frame]`。  
- 本地方法栈`[Native Stack]`： 主要用于执行其他语言的方法。  
- 堆`[Heap]`：用于存放对象实例。  
- 方法区`[Method Area]`： 存放已被虚拟机加载的类信息、常量、静态变量等。  


## 垃圾回收算法  

主要包含以下三种算法：  

- `标记清除算法`  

    - 描述： 标记所有需要回收的对象，然后将其清除。  
    - 优点： 简单易懂  
    - 缺点： [1]标记、清除两个过程效率均不高，[2]内存碎片率高。  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/mark-sweep.png)  


- `复制算法`  

    - 描述： 将内存按照容量划分为A、B两块区域，将存活的对象赋值到B区域中，然后清理A区域。  
    - 优点： [1]效率高于标记-清除算法，[2]内存碎片率低。  
    - 缺点： 需要浪费约10%的内存。  
    - 应用场景： 对象存活率低。  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/copying.png)  


- `标记整理算法`  

    - 描述： 标记所有需要回收的对象，然后所有存活的对象都往一端移动，最后清理边界以外的内存。  
    - 优点： 内存碎片率低。  
    - 缺点： 效率没有复制算法高。  
    - 应用场景： 对象存活率高。

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/mark-compact.png)  


## GC垃圾收集器  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/JVM_garbage_collectors.png)  

## 内存分配策略  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/JVM_memory_allocate_strategy.png)  

这里说明下：什么情况下部分对象会直接进入老年代？  

当`Survivor`空间中某一年龄的对象大于该空间的一半，那么大于等于该年龄段的对象直接进入老年代。  


## `Reference`  
- `《深入理解Java虚拟机》 - 第二章[Java内存区域与内存溢出异常]`  
- `《深入理解Java虚拟机》 - 第三章[垃圾收集器与内存分配策略]`  