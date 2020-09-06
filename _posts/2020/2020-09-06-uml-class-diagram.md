---
layout: post
title: UML - 类图
category: 
tags: []
excerpt: UML - Class Diagram
---

本文总结一下`UML`类图的用法，主要包含基本结构、可见性、类关系和多重性，如图：  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/UML_class_diagram.png)  


## Basic    

首先，我们先来看下类图的定义：  

> In software engineering, a class diagram in the Unified Modeling Language (UML) is a type of static structure diagram that describes the structure of a system by showing the system's classes, their attributes, operations (or methods), and the relationships among objects.  

`类图`是`UML`中的一种`静态结构图`，它的作用是通过展示系统中的类、及其成员变量和方法、对象之间的关系来描述整个系统的结构。  

然后，我们来看下`类图`的基本结构：  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/class_diagram_demo.png)  

主要由以下三部分组成：  

- 类名  
- 类成员变量  
- 类方法  


## Visibility    

在上图中，成员变量和方法的最左侧有`-`和`+`符号，那么它们有什么含义呢？  

它们其实指定了类成员的可见性，与`Java`类似，主要有以下四种：  

- `~`: Package / Default  
- `+`: Public  
- `-`: Private  
- `#`: Protected  


## Relationships    

再来看下类之间的关系：   

### Inheritance  

表示两个类拥有继承关系：  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/inheritance.png)  


### Association  

表示类之间的存在某种关系，如：航班与飞机、乘客与地铁等。  


![](https://yyc-images.oss-cn-beijing.aliyuncs.com/association.png)  


### Aggregation  

`Aggregation`是`Association`中更为具体的一种表示方式。   

当一个类作为其他类的集合或容器时，就会发生`Aggregation`，   

但是被包含的类的生命周期并不会依赖于该容器，所以即使容器类被销毁，被包含的类仍存在。   

比如： 图书馆与书，即使图书馆不存在了(翻修)，书仍可以存放在仓库中。   

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/aggregation.png)  


### Composition    

`Composition`与`Aggregation`类似，区别是：  

当容器类被销毁时，它的被包含的类也会销毁。  

比如： 一间房子中有卧室、厨房、阳台等，一旦房子被拆，那么其组成部分均被销毁。  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/composition.png)  


## Multiplicity    


最后介绍下类与类之间的数量匹配关系，一般有以下五种：  

- `0..1`: 零或一  
- `n`: 固定数量   
- `0..*`: 零或多个  
- `1..*`: 一或多个  
- `m..n`: 指定范围    

比如：   

- 一笔订单只能有一笔订单详情记录。  
- 一个用户可以有零或多笔订单。  



## Reference    
- [UML Class Diagram Tutorial](https://www.youtube.com/watch?v=UI6lqHOVHic){:target="_blank"}  
- [Class diagram](https://en.wikipedia.org/wiki/Class_diagram){:target="_blank"}  
- [UML Class Diagram Relationships Explained with Examples](https://creately.com/blog/diagrams/class-diagram-relationships/){:target="_blank"}  
- [lucidchart](https://app.lucidchart.com/){:target="_blank"}  


