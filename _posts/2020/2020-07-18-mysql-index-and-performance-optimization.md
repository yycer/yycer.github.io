---
layout: post
title: MySQL - 索引与性能优化
category: database
tags: [database]
excerpt: Index and Performance Optimization
---

Thanks 

[Jeffery Zhang](https://www.youtube.com/watch?v=BbKAzDJ2H9A&list=PLkEGzwfkhPnTfUMs_-URN5c130zM2mruK&index=2&t=7098s){:target="_blank"}  


## 索引数据结构推演  

### 哈希  

它的优点是什么？  

查询的时间复杂度为`O(1)`  

那么缺点是什么？  

第一，可能会发生`哈希碰撞`。  

第二，无法计算区间。   


### BST  

来看下示意图：  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/bst_common.png)  

它的时间复杂度为`O(logn)`。  

致命缺点在于： 极端情况下会退化为单链表。  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/bst_special.png)  


### 平衡二叉搜索树    

`平衡二叉搜索树`保证了左右子树的平衡，从而保证了`O(logn)`的时间复杂度。  

来看下示意图：  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/bst_balance.png)  

注意它的数据结构，包含：`关键字`、`数据区`和`左右子节点的引用`。  

那么它的缺点是什么呢？  

第一，树的高度太高，导致与磁盘交互的`IO`次数太多，效率低。  


第二，每次与磁盘交互的性价比极低。  


因为，操作系统与磁盘以`页`为交互单位，默认为`4KB`，`MySQL`调整为`16KB`，  

但是，目前阶段，我们只接收了极少量的数据。  


### B树  

`B树`正是改进了上述两个缺点，  

假设一个结点`(关键字+数据区+子结点引用)`占`8B`，那么一次磁盘交互返回的数据为`16KB`，  

我们当然期望能一次拿到约`2000`个结点，然后放在内存中进行比较。   

来看下示意图：  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/b_tree.png)  


那么`B树`的缺点是什么呢？  

不够稳定，可能遍历一次就拿到需要的结果了，也有可能需要遍历`2、3`次。  


### B+树  

来看下示意图：  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/b_plus_tree.png)  


它有如下几个特点：  

第一，每个结点关键的区间为`左闭右开`。  

第二，数据区的数据都保存在叶子节点，并且彼此连接，呈升序排列。  

好处在于，查询时间稳定，因为最终的数据你都要去叶子节点获取。  

同时，具有天然强大的`排序`能力。  



## 性能优化  


### 索引命中  

首先，我们需要知道`离散度`这个概念。  

如果同一列中的数据重复率越低，说明它的离散度越高。  

相反，比如：性别，它的离散度就很低。   

回想一下`B+`树的结构，如果离散度很低，那么其中的结点，它的路径选择性就很差，  

因为下一层的结点可能都是重复的。  

所以，我们要避免在离散度很低的列上创建索引。  

第二点，是`最左匹配原则`。  

它的含义是： 索引项中`关键字`的对比，一定是`从左到右`进行的。  

举个例子：  

`select * from users where name = '%xxx';`  

对于第一个字符来说，它可以匹配任何字符，所以可能不会用到索引，甚至变成全表扫描。  

还有个经典的例子就是`联合索引`，举个例子：  

``` 
联合索引: users(name, age, phone)  

select * from users
where phone = 'xx' and name = 'xx';
```

此时，根本就不会命中这个联合索引。   

当然，如果你使用范围查询，即使满足最左匹配，也只会命中一部分：  

```
select * from users
where name = 'yyc' and age > 20 and phone = '133';
```



### 聚集/覆盖索引  

首先，我们需要明确一下`MyISAM`和`InnoDB`两种数据引擎对于索引和真实数据的存储区别。  

对于`MyISAM`来说，它有一个独立的文件存储索引结构，并且所有的索引都是平级的，  

在对应`B+`树的叶子节点上存储的是指向具体数据的引用。   

而具体的数据会保存在另一个单独的文件中。  

以下示意图来自视频教学中。   

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/myisam_index_data.png)  


但是，对于`InnoDB`引擎来说，主键索引的等级最高，  

在主键索引`B+`树的叶子节点中，存储的就是具体的数据。  

而其他类型的索引，如：普通索引、唯一索引、联合索引它们的叶子节点存储的是主键的关键字。  

所以，一旦它们需要查询其他的数据，就要依赖这个主键关键字，  

然后到对应的主键`B+`树中找到对应的数据，这个过程称为`回表`。  

一旦发生了回表，那么就会消耗更多的查询时间，所以我们要避免使用通配符`*`。  

所以，简单来说聚集索引就是主键，因为它包含具体的数据，  

而覆盖索引，就是查询过程中，没有发生`回表`。  

来看几个例子：  

哪些语句没有发生回表查询？  

```
索引: PK(id), key(name, phone) unique(userNum)

1. select userNum from teacher where userNum = ?;
2. select * from teacher where name = ?;
3. select id, userNum from teacher where userNum = ?;
4. select name, phone from teacher where userNum = ?;
5. select phone from teacher where name = ?;
```


`1、3、5`  
