---
layout: post
title: MySQL - autocommit模式与异常处理
category: database
tags: [database]
excerpt: Autocommit Mode, Error Handling
---

## Autocommit Mode  

- 如何查看`autocommit`模式的状态？  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/autocommit_1_select.png)


- 如何修改`autocommit`模式的状态？  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/autocommit_1_set.png)

- `commit`的含义是什么？  

> A COMMIT means that the changes made in the current transaction are made permanent and become visible to other sessions.  

`Commit`意味着当前事务中所有改动都是永久性的，对其他会话都是可见的。  

- 如何理解永久性？  

这就涉及到`ACID`中的`Durability`(持久性)。当我们往表中`insert`一条记录时，默认情况下`autocommit`模式是开启的，所以默认做了`commit`动作，因此插入的数据保存到了`MySQL`服务器，同时对其他会话`session`可见。如果我们把`autocommit`关闭了会怎么样？  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/autocommit_set_0.png)

结合3和4可以看出，当`autocommit = 0`时，在`session1`(左边)往`account`表中插入一条新记录后，在`session2`(右边)是不可见的。  

结合5和6可以看出，当我们人为`commit`或`rollback`后，当前会话中修改的数据才会对其他会话可见。  


- `autocommit`模式官方描述  

> In InnoDB, all user activity occurs inside a transaction.  
> If autocommit mode is enabled, each SQL statement forms a single transaction on its own.  
> By default, MySQL starts the session for each new connection with autocommit enabled,  
> so MySQL does a commit after each SQL statement if that statement did not return an error.  
> If a statement returns an error, the commit or rollback behavior depends on the error.

对于`InnoDB`存储引擎，所有的用户操作都是发生在事务中的。  
一旦`autocommit`模式开启，每一段`SQL`语句都是一个单一的事务。  
默认情况下，每开启一个`MySQL`连接，对应会话中的`autocommit`模式都是开启的。  
一旦语句中发生错误，就会根据错误的类型决定是否部分提交、还是全部回滚。



<br>
## InnoDB Error Handling  

接下来详细说明下，当事务发生异常时，针对不同类型的异常错误，`InnoDB`存储引擎是如何处理的。  

- `Transaction deadlock`(事务死锁)  

> A transaction deadlock causes InnoDB to roll back the entire transaction.  

注意喔，发生事务死锁时，它会回滚整个事务。  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/error_1_transaction_deadlock_rollback_all.png)

<br>

- `Lock wait timeouts`(锁等待超时)

> A lock wait timeout causes InnoDB to roll back only the single statement that was waiting for the lock and encountered the timeout.  

当发生锁等待超时错误时，只回滚发生异常的语句。  

那么超时的时间默认是多久呢`MySQL 8.0`？  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/error_2_innodb_lock_wait_timeout_default_50.png)

我们将超时的时间设置短一点，模拟下锁等待超时的场景。  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/error_2_lock_wait_timeout_rollback_part.png)

<br>

- `duplicate-key error`(重复键错误)  

> A duplicate-key error rolls back the SQL statement.

重复键错误，也只会回滚发生异常的`SQL`语句。

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/error_3_duplicate_keys_error_rollback_part.png)

<br>

- `row-too-long error`(行值过长错误)

> A row too long error rolls back the SQL statement.

行值过长错误，也一样，只回滚发生异常的语句。  


![](https://yyc-images.oss-cn-beijing.aliyuncs.com/error_4_row_too_long_error_rollback_part.png)


<br>
## 参考  

- [15.7.2.2 autocommit, Commit, and Rollback](https://dev.mysql.com/doc/refman/8.0/en/innodb-autocommit-commit-rollback.html){:target="_blank"}
- [15.20.4 InnoDB Error Handling](https://dev.mysql.com/doc/refman/8.0/en/innodb-error-handling.html){:target="_blank"}
- [15.13 InnoDB Startup Options and System Variables](https://dev.mysql.com/doc/refman/8.0/en/innodb-parameters.html#sysvar_innodb_lock_wait_timeout){:target="_blank"}
