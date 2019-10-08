---
layout: post
title: MySQL - 一致性读
category: database
tags: [database]
excerpt: Consistent Read
---

> 以下所有实验使用的版本为: `MySQL 8.0`

## 一致性读是什么？  

> A consistent read means that InnoDB uses multi-versioning to present to a query a snapshot of the database at a point in time.  
> The query sees the changes made by transactions that committed before that point of time, and no changes made by later or uncommitted transactions.

一致性读意味着`InnoDB`存储引擎使用多版本控制的方式显示某一时刻的数据库查询快照。  
查询的结果是该时间节点之前提交的事务所做的修改，以后的就看不到了。

<br>
## 如何触发一致性读？  

> Consistent read is the default mode in which InnoDB processes SELECT statements in READ COMMITTED and REPEATABLE READ isolation levels.

很明显，当你使用`InnoDB`存储引擎，同时你的事务隔离级别是`READ COMMITTED`或`REPEATABLE READ`时，所有`SELECT`语句返回的结果都是一致性读。  

<br>
## 一致性读有什么特殊场景？  

> The exception to this rule is that the query sees the changes made by earlier statements within the same transaction. 

虽然说一致性读获得是某个时间节点对于数据库的查询快照，但在同一个事务内部，查询快照还是会发生改变的，我们来看个例子。  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/consistent_read_1_read_changes_within_transaction.png)

<br>
## RR和RC隔离等级下的一致性读  

接下来，我们来看下`REPEATABLE READ`和`READ COMMITTED`事务隔离等级下关于一致性读的几种场景。  

- RR与RC隔离等级下，关于一致性读的区别是什么？  

当事务A、B均处于`REPEATABLE READ`隔离等级，若A做了数据修改，等A提交后，B仍查询不到修改的数据。  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/consistent_read_2_RR_no_changes_before_both_commit.png)

可以在`第9步`看到，只有当`Session1`和`2`都提交后，才会`Session2`中查询到之前`Session1`中插入的新数据。  

<br>

当事务A处于`REPEATABLE READ`，事务B处于`READ COMMITTED`事务等级时，若A做了数据修改，等A提交后，B是可以查询到修改的数据！  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/consistent_read_3_RR_RC_session2_get_changes_after_session1_commit.png)

可以看到，当`Session1`在`第7步`提交后，`Session2`在`第8步`读取到了新的添加数据。  


所以，总结下两种事务隔离等级下，关于一致性读的区别：  

> 当事务A、B均为`REPEATABLE READ`等级时，当A修改完数据并提交后，B仍读取不到最新的数据。  
> 当事务A是`REPEATABLE READ`，B是`READ COMMITTED`等级时，当A修改完数据并提交后，B可以读取到最新的数据。  

<br>

- 那么在RR隔离等级下，如何获得最新的查询快照？  

第一种方式: `Share Lock(共享锁)`  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/consistent_read_4_RR_has_changes_use_S_lock.png)

<br>
第二种方式: `Exclusive Lock(排它锁)`  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/consistent_read_5_RR_has_changes_use_X_lock.png)

<br>
第三种方式: `UPDATE, DELETE`  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/consistent_read_6_RR_has_changes_use_Update.png)

<br>
## 参考  

- [15.7.2.3 Consistent Nonlocking Reads](https://dev.mysql.com/doc/refman/8.0/en/innodb-consistent-read.html){:target="_blank"}