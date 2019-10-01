---
layout: post
title: MySQL中存储过程与函数的使用
category: database
tags: [database]
excerpt: stored-procedure, function
---
![](https://yyc-images.oss-cn-beijing.aliyuncs.com/mysql-procedure-function.png)
## 准备数据  

``` sql
CREATE TABLE IF NOT EXISTS orders(
    id       BIGINT auto_increment,
    order_id VARCHAR(50),
    user_id  VARCHAR(20),
    total    DECIMAL not null,
    created_date datetime,
    PRIMARY KEY (id)
);

INSERT INTO orders(order_id, user_id, total, created_date)
VALUES('4c841716-e28b-11e9-8960-00ff7b45daf1', '20004007596', 200.00, '2019-09-03 08:10:00');
INSERT INTO orders(order_id, user_id, total, created_date) 
VALUES('4c841716-e28b-11e9-8960-00ff7b45daf2', '20004007588', 300.00, '2019-10-03 15:10:00');
INSERT INTO orders(order_id, user_id, total, created_date) 
VALUES('4c841716-e28b-11e9-8960-00ff7b45daf3', '20004007539', 400.00, '2019-09-28 13:10:00');
INSERT INTO orders(order_id, user_id, total, created_date) 
VALUES('4c841716-e28b-11e9-8960-00ff7b45daf4', '20004007542', 300.00, '2019-09-22 21:10:00');
INSERT INTO orders(order_id, user_id, total, created_date) 
VALUES('4c841716-e28b-11e9-8960-00ff7b45daf5', '20004007501', 600.00, '2019-10-30 12:10:00');

```

<br>
## 存储过程  

> 如何创建一个存储过程？  

``` sql
CREATE PROCEDURE uspGetOrders()
BEGIN
    SELECT order_id, user_id, total, created_date FROM orders;
END
```

> 如何创建一个包含输出参数的存储过程？  

``` sql
CREATE PROCEDURE uspGetOrderById(
    userId BIGINT,
    OUT rowId INT)
BEGIN
    SELECT id INTO rowId FROM orders
    WHERE user_id = userId;
END

```

> 如何执行存储过程？  

``` sql
CALL uspGetOrders();

CALL uspGetOrderById(20004007596, @id);
SELECT @id; # 1
```
![](https://yyc-images.oss-cn-beijing.aliyuncs.com/mysql-procedure-function.png)

> 如何修改存储过程？  

第一种方式是借助工具实现，如: `Navicat`。  

第二种是官方文档提供的:
> However, you cannot change the parameters or body of a stored procedure using this statement; to make such changes, you must drop and re-create the procedure using DROP PROCEDURE and CREATE PROCEDURE.  

大致意思就是说，你不能用`ALTER PROCEDURE proc_name [characteristic ...]`语句改变存储过程的内容，如果你实在要做调整，就只能先删再创建。

那上述语句有什么作用呢？  

首先需要了解下`characteristic`包含哪些特征。  

```
characteristic:
    COMMENT 'string'
  | LANGUAGE SQL
  | { CONTAINS SQL | NO SQL | READS SQL DATA | MODIFIES SQL DATA }
  | SQL SECURITY { DEFINER | INVOKER }
```

第一个特征表示该存储过程的备注信息。  

第二个特性说明该存储过程是由`SQL`语句组成。  

第三个特性具体说明语句的组成。`READS SQL DATA`代表仅包含查询语句，`MODIFIES SQL DATA`代表包含写数据的语句。  

第四个特性代表执行该存储过程的权限。`DEFINER`表示，存储过程中涉及的某些表，只要该存储过程的创建者有权限执行，那任意调用者均有权限执行。`INVOKER`表示，根据调用者对于某些表的权限，来判断是否有权限执行该存储过程。  


> 如何查看某个存储过程的内容？

``` sql
SHOW CREATE PROCEDURE procedure_name;
```
![](https://yyc-images.oss-cn-beijing.aliyuncs.com/getProcedureContent.png)

> 如何删除某个存储过程？

``` sql
DROP PROCEDURE IF EXISTS procedure_name;
```

> 如何查看一张表的结构？ 

``` sql
DESCRIBE table_name;
```
![](https://yyc-images.oss-cn-beijing.aliyuncs.com/describeOrder.png)

<br>
## 函数  

> 如何创建函数？  

``` sql
CREATE FUNCTION Pascal(
    `name` VARCHAR(50)
)
RETURNS VARCHAR(50)
DETERMINISTIC -- 保证函数幂等性
    DECLARE processed_name VARCHAR(50);
    SET processed_name = CONCAT(UPPER(LEFT(`name`, 1)), SUBSTRING(`name`, 2, LENGTH(`name`)));
    RETURN (processed_name);
END
```

> 如何执行某个函数？  

``` sql
SELECT Pascal('frankie'); # Frankie
```
> 如何修改某个函数？  

同存储过程。

> 如何查看某个函数的内容？ 

``` sql
SHOW CREATE FUNCTION function_name;
```
![](https://yyc-images.oss-cn-beijing.aliyuncs.com/getFunctionContent.png)

> 如何删除某个函数？  

``` sql
DROP FUNCTION IF EXISTS function_name;
```

<br>
## 存储过程与函数的区别？  

- 结构层面: 存储过程可以包含输入、出参数，而函数仅支持输入参数，并且只能通过`return`方式输出。
- 功能层面: 存储过程更适合存储独立、较为复杂的`SQL`语句，而函数可以作为`SQL`语句的一部分，比如计算总数`count()`, 求和`sum()`等。

<br>
## 参考

- 《MySQL必知必会》 第二十三章 - 使用存储过程
- [ALTER PROCEDURE Syntax](https://dev.mysql.com/doc/refman/8.0/en/alter-procedure.html)
- [Mysql中创建存储过程和函数的语法](http://www.hello-code.com/blog/mysql/201602/5870.html)