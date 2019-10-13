---
layout: post
title: MySQL - 幻读
category: database
tags: [database]
excerpt: Phantom Read
---

## What is Phantom Read?  

这几天在网上看了大量相关文章，关于是否会在`Repeatable Read`事务隔离等级下产生幻读问题，众说纷纭。之前只是仿照他人写了个小例子，并没有深入了解，因此决定单独写篇文章，结合官方文档以及其他博客重新梳理下。  

首先第一个问题就是，什么是`Phantom Read`?

> The so-called phantom problem occurs within a transaction when the same query produces different sets of rows at different times.  

也就是说，在一个事务内部，相同的查询语句，执行多次的返回结果不同。  

这里需要补充两点，  
第一，我们假设在`Repeatable Read`事务隔离等级下。  
第二，上面提到查询语句是普通查询`Plain Query`，也就是`Non-locking Read`。

<br>
下面我们来看个例子：

<br>
## Example  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/phantom_read.png)


首先，看第五步，在`Session2`中往`user`表中插入一条新的数据。  
然后，回到`Session1`中的第六步，并没有查询出新插入的数据，因为在`RR`隔离等级下，使用的是`Consistent Read`。  
接下来，第七步，`Session2`提交事务。  
第八步，`Session1`仍没有查询到新插入的数据。  
最后，第九步，在`Session1`中更新一条看似不存在，但实际在`Session2`  中新插入的数据，然后相同的查询语句，这次查询到了新的数据，此时使用的是`Current Read`。


<br>
## How to avoid that?  

那么如何避免`Phantom Read`?  

第一种方式: `Locking Read`  

`Phantom Read`是因为在同一个事务中多次执行相同的查询语句，返回的结果不同，也就是说会返回其他事务新插入的数据。  

那么在并发的场景下，想办法阻止其他事务插入新数据不就行了。  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/phantom_read_avoid_1.png)

在第三步中对`user`表进行`Locking Read`，因此在第四步，当`Session2`试图往`user`表中插入一条新数据时，被阻塞`waiting`。  

<br>
第二种方式: 使用`Serializable`隔离等级，不过在高并发场景下，性能太差，不建议使用。  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/phantom_read_avoid_2.png)

可以看到，在`Serializable`隔离等级下，可以阻止`Phantom Read`的关键在于，一旦在其他事务中插入新的数据，回到当前事务中你永远也无法读到。  
因为在当前情况下，你只能`Consistent Read`，也就是读取开启当前事务时的查询快照`SnapShot`，一旦使用`Current Read`，比如`Locking Read - Select for share, update`，或者`DML(Data Manipulation Language) - Insert, Update, Delete`就会被阻塞。

<br>
## Reference  
[15.7.4 Phantom Rows](https://dev.mysql.com/doc/refman/8.0/en/innodb-next-key-locking.html){:target="_blank"}  
[複習資料庫的 Isolation Level 與圖解五個常見的 Race Conditions](https://medium.com/@chester.yw.chu/%E8%A4%87%E7%BF%92%E8%B3%87%E6%96%99%E5%BA%AB%E7%9A%84-isolation-level-%E8%88%87%E5%B8%B8%E8%A6%8B%E7%9A%84%E4%BA%94%E5%80%8B-race-conditions-%E5%9C%96%E8%A7%A3-16e8d472a25c){:target="_blank"}  
[對於 MySQL Repeatable Read Isolation 常見的三個誤解](https://medium.com/@chester.yw.chu/%E5%B0%8D%E6%96%BC-mysql-repeatable-read-isolation-%E5%B8%B8%E8%A6%8B%E7%9A%84%E4%B8%89%E5%80%8B%E8%AA%A4%E8%A7%A3-7a9afbac65af){:target="_blank"}  
[Bug #63870 Repeatable-read isolation violated in UPDATE](https://bugs.mysql.com/bug.php?id=63870){:target="_blank"}  
[Testing MySQL isolation levels](https://medium.com/@huynhquangthao/mysql-testing-isolation-levels-650a0d0fae75){:target="_blank"}  
[Understanding MySQL Isolation Levels: Repeatable-Read](https://blog.pythian.com/understanding-mysql-isolation-levels-repeatable-read/){:target="_blank"}
