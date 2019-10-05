---
layout: post
title: MySQL - 事务隔离等级
category: database
tags: [database]
excerpt: Transaction Isolation LEVEL
---


![](https://yyc-images.oss-cn-beijing.aliyuncs.com/transaction_0.png)


## 准备数据  

``` sql
CREATE TABLE IF NOT EXISTS orders(
    id       BIGINT auto_increment,
    order_id VARCHAR(50),
    user_id  VARCHAR(20),
    total    DECIMAL not null,
    created_date datetime,
    PRIMARY KEY (id),
    UNIQUE key (order_id)
);

CREATE TABLE `account` (
  `id`      int(11)      NOT NULL AUTO_INCREMENT,
  `name`    varchar(255) DEFAULT NULL,
  `balance` int(11)      DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE = InnoDB AUTO_INCREMENT = 4 DEFAULT CHARSET = utf8

INSERT INTO `account` (`name`, `balance`) VALUES ('lilei',  '450');
INSERT INTO `account` (`name`, `balance`) VALUES ('hanmei', '16000');
INSERT INTO `account` (`name`, `balance`) VALUES ('lucy',   '2400');
```

<br>
## 如何创建并执行事务？  

- 事务是什么？  

> A transaction is a sequential group of database manipulation operations, which is performed as if it were one single work unit.

事务是一组顺序执行的数据库操作，它们被当做一个单一的工作单元执行。

- 事务有什么特征？  

> A transaction will never be complete unless each individual operation within the group is successful. If any operation within the transaction fails, the entire transaction will fail.

也就是说只有当事务中的所有操作都成功时，这个事务才算执行成功，否则，有任意一个操作失败，整个事务也就执行失败了。

- 如何创建并执行事务？  


``` sql
START TRANSACTION;
INSERT INTO orders(order_id, user_id, total, created_date)
VALUES('4c841716-e28b-11e9-8960-00ff7b45daf1', '20004007588', 200.00, '2019-09-27 08:10:00');
INSERT INTO orders(order_id, user_id, total, created_date)
VALUES('4c841716-e28b-11e9-8960-00ff7b45daf2', '20004007585', 500.00, '2019-10-02 15:10:00');
COMMIT;
```

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/transaction_1_create_and_invoke_simple_transaction.png)

- 模拟事务失败的场景  

由于`orders`表对`order_id`列建立了唯一索引`unique index`，因此当插入包含重复`order_id`的记录时，就会失败。  

``` sql
START TRANSACTION;
INSERT INTO orders(order_id, user_id, total, created_date)
VALUES('4c841716-e28b-11e9-8960-00ff7b45daf1', '20004007589', 120.00, '2019-10-05 11:10:00');
INSERT INTO orders(order_id, user_id, total, created_date)
VALUES('4c841716-e28b-11e9-8960-00ff7b45daf3', '20004007572', 350.00, '2019-10-26 13:10:00');
COMMIT;
```

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/transaction_2_rollback.png)
<br>

因此，`orders`表中仍只有两条记录。  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/transaction_1_create_and_invoke_simple_transaction.png)


<br>
## 事务隔离等级  

首先讲述下一些基本的操作  

- 一共包含哪几种事务隔离等级？  

`Read Uncommitted`(读未提交)  
`Read Committed`(读已提交)  
`Repeatable Read`(可重复读)  
`Serializable`(可串行化)  

现在一头雾水没有关系，后面会对这四种等级做详细的介绍与分析。


- 如何查看事务隔离等级？  

``` sql
SELECT @@SESSION.TRANSACTION_ISOLATION;
```

- 如何设置事务隔离等级？  

``` sql
SET SESSION TRANSACTION ISOLATION LEVEL REPEATABLE READ;
```

- `Read Uncommitted`  

这是最低的事务隔离级别，它代表在事务A内部可以读取事务B修改但尚未提交的数据。但这会导致脏读`dirty read`问题。  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/transaction_3_dirty_read.png)

- `Read committed`  

该事务隔离等级解决了脏读的问题，也就是说，只有当事务B提交后，在事务A内部才能获得更改后的数据。但是，这又引发了另一个问题，也就是`不可重复读`。当事务B还没提交前，无论如何修改数据，在事务A内部数据都是不变的，一旦事务B提交后，在事务A内部读取数据，就会发生前后读取的数据不一致情况。  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/transaction_4_non_repeatable_read.png)

- `Repeatable Read`  

该事务隔离等级解决了不可重复读的问题，它的做法是，在事务A提交之前，无论事务B如何修改数据，在事务A内部读取的数据都是一样的(其实是使用了快照`snapshot`)。但这也会有个问题，假设事务B插入了一条新的数据，因为事务A内部使用的是快照，因此没有读取到新的数据，但是在事务A内部却可以影响到该条数据，比如: `UPDATE`，这就是所谓的幻读`phantom read`。    


![](https://yyc-images.oss-cn-beijing.aliyuncs.com/transaction_5_phantom_read.png)


- `Serializable`  

这是最高级的事务隔离等级，虽然准确性很高，但是在高并发场景下，效率很低，因此不建议使用。  


![](https://yyc-images.oss-cn-beijing.aliyuncs.com/transaction_6_serializable.png)

<br>
## 参考  

- [深入理解MySQL锁与事务隔离级别](https://www.gaoyaqiu.com/post/mysql/mysql-lock-transaction/#%E8%AF%BB%E6%9C%AA%E6%8F%90%E4%BA%A4-read-uncommitted){:target="_blank"}
- [Mysql锁：灵魂七拷问](https://tech.youzan.com/seven-questions-about-the-lock-of-mysql/){:target="_blank"}
- [Testing MySQL isolation levels](https://medium.com/@huynhquangthao/mysql-testing-isolation-levels-650a0d0fae75){:target="_blank"}