---
layout: post
title: MySQL - 触发器
category: database
tags: [database]
excerpt: 深入理解MySQL触发器
---

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/1_trigger.png)

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
```

<br>
## 如何创建触发器？  

- 首先触发器是什么？  

> A trigger is a named database object that is associated with a table, and that activates when a particular event occurs for the table.

可以把触发器理解成一个数据库对象，它与一张表关联，当该表发生某些特定行为，如`insert, update, delete`时，它被触发。  

- 触发器有哪些部分组成的？

``` sql
CREATE TRIGGER ins_sum BEFORE INSERT ON orders
FOR EACH ROW
SET @sum = @sum + NEW.total;
```
`ins_sum`代表触发器名称`Tigger name`。  
`BEFORE`代表触发器触发时机`action time`，还有`AFTER`。  
`INSERT`代表触发器事件类型`Tigger event`，还有`UPDATE`和`DELETE`。  
`orders`代表触发器应用的相关表`related table`。  
剩下的代表触发器内容`Tigger body`。  

- 那么如何执行一个触发器呢？

``` sql
SET @sum = 0;
INSERT INTO orders(order_id, user_id, total, created_date)
VALUES('4c841716-e28b-11e9-8960-00ff7b45daf6', '20004007533', 100.00, '2019-09-03 08:10:00');
SELECT @sum as 'total amount inserted';
```

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/2_simple_tigger_activate_result.png)

- 能否对同一张表创建多个相同事件类型、触发时间的事件？

> It is possible to define multiple triggers for a given table that have the same trigger event and action time.

很明显，这是有可能的。

- 那么如何保证多个触发器之间的执行顺序？

> By default, triggers that have the same trigger event and action time activate in the order they were created.

也就是说，默认情况下，谁先创建，谁先执行。

``` sql
CREATE TRIGGER ins_first BEFORE INSERT on orders
FOR EACH ROW SET @str = 'first';

CREATE TRIGGER ins_second BEFORE INSERT on orders
FOR EACH ROW SET @str = 'second';

SET @str = '';
INSERT INTO orders(order_id, user_id, total, created_date)
VALUES('4c841716-e28b-11e9-8960-00ff7b45daf6', '20004007533', 100.00, '2019-09-03 08:10:00');
SELECT @str;

```
因此`first`被`second`覆盖。  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/3_tigger_default_activate_sequence.png)

- 如果我要人为控制触发器的执行顺序，应该怎么办？  

> To affect trigger order, specify a clause after FOR EACH ROW that indicates FOLLOWS or PRECEDES and the name of
an existing trigger that also has the same trigger event and action time.  
With FOLLOWS, the new trigger 
activates after the existing trigger.  
With PRECEDES, the new trigger activates before the existing trigger.

`MySQL`提供两种方式: `FOLLOWS`和`PRECEDES`。假设我们已经创建了触发器A，那么当我们使用`FOLLOWS`时，新建的触发器将在A之后触发，当我们使用`PRECEDES`时，新建的触发器将在A之前触发。  

``` sql
DROP TRIGGER ins_second;
SHOW TRIGGERS in test;

CREATE TRIGGER ins_third BEFORE INSERT on orders
FOR EACH ROW PRECEDES ins_first
SET @str = 'third';
SHOW TRIGGERS in test;

SET @str = '';
INSERT INTO orders(order_id, user_id, total, created_date)
VALUES('4c841716-e28b-11e9-8960-00ff7b45daf6', '20004007533', 100.00, '2019-09-03 08:10:00');
SELECT @str;
```

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/4_tigger_precedes.png)

- 接下来，我们来讲讲`Tigger body`中的`OLD`, `NEW`关键字。

> In an INSERT trigger, only NEW.col_name can be used; there is no old row.   
In a DELETE trigger, only OLD.col_name can be used; there is no new row.   
In an UPDATE trigger, you can use OLD.col_name to refer to the columns of a row 
before it is updated and NEW.col_name to refer to the columns of the row after it is updated.

`UPDATE`比较特殊，只有它才能同时使用`OLD`和 `NEW`。  

``` sql
SHOW TRIGGERS in test;
DROP TRIGGER ins_first;
DROP TRIGGER ins_third;

CREATE TRIGGER update_show_old_and_new_values BEFORE UPDATE on orders
FOR EACH ROW
SET @old_value = OLD.total,
    @new_value = NEW.total;
    
SHOW TRIGGERS in test;

SET @old_value = '', @new_value = '';
UPDATE orders SET total = 200.00 WHERE order_id = '4c841716-e28b-11e9-8960-00ff7b45daf6';
SELECT @old_value, @new_value;
```

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/5_update_tigger_show_old_and_new_value.png)

继续深入理解下`BEFORE trigger`  

> A column named with OLD is read only.  
> You can refer to it (if you have the SELECT privilege), but not modify it.  
You can refer to a column named with NEW if you have the SELECT privilege for it.  
In a BEFORE trigger, you can 
also change its value with SET NEW.col_name = value if you have the UPDATE privilege for it. This means you can 
use a trigger to modify the values to be inserted into a new row or used to update a row. (Such a SET statement 
has no effect in an AFTER trigger because the row change will have already occurred.)

主要是说，当触发时机是`BEFORE`时，在触发器体`Tigger body`内可以修改某列的值，但如果是`AFTER`时，你就不能修改了。那么问题来了，如果在`BEFORE`时修改了某列的值，并且在`insert`或`update`对该列也做了调整，那么最终结果以谁为准？  

理所当然，我猜是后者。

``` sql
SHOW TRIGGERS in test;
DROP TRIGGER update_show_old_and_new_values;

CREATE TRIGGER ins_before_set_new_value BEFORE INSERT on orders
FOR EACH ROW SET NEW.total = 200.00;
SHOW TRIGGERS in test;

INSERT INTO orders(order_id, user_id, total, created_date)
VALUES('4c841716-e28b-11e9-8960-00ff7b45daf6', '20004007533', 100.00, '2019-09-03 08:10:00');

SELECT * from orders where order_id = '4c841716-e28b-11e9-8960-00ff7b45daf6'

```
![](https://yyc-images.oss-cn-beijing.aliyuncs.com/6_insert_tigger_before_set_new_value.png)

``` sql
SHOW TRIGGERS in test;
DROP TRIGGER ins_before_set_new_value;

CREATE TRIGGER update_before_set_new_value BEFORE UPDATE on orders
FOR EACH ROW SET NEW.total = 1000.00;
SHOW TRIGGERS in test;

UPDATE orders SET total = 500.00 WHERE order_id = '4c841716-e28b-11e9-8960-00ff7b45daf6';
SELECT * from orders where order_id = '4c841716-e28b-11e9-8960-00ff7b45daf6'

```
![](https://yyc-images.oss-cn-beijing.aliyuncs.com/7_update_tigger_before_set_new_value.png)

``` sql
SHOW TRIGGERS in test;
DROP TRIGGER update_before_set_new_value;

CREATE TRIGGER insert_after_set_new_value AFTER INSERT on orders
FOR EACH ROW SET NEW.total = 333.00;

CREATE TRIGGER update_after_set_new_value AFTER UPDATE on orders
FOR EACH ROW SET NEW.total = 555.00;
```
![](https://yyc-images.oss-cn-beijing.aliyuncs.com/8_update_new_column_is_not_allowed_in_after_tigger.png)

看了上面三个例子，事实证明，我的猜测是错误的。

- 还有一点是关于自增字段的。

> In a BEFORE trigger, the NEW value for an AUTO_INCREMENT column is 0, not the sequence number that is generated 
automatically when the new row actually is inserted.

这点很简单，就是说当触发时机时`BEFORE`时，对于自增字段，`NEW.AUTO_INCREMENT_COLUMN`的值为0。

``` sql
CREATE TRIGGER insert_get_auto_increment BEFORE INSERT on orders
FOR EACH ROW SET @num = NEW.id;
SHOW TRIGGERS in test;

SET @num = 0;
INSERT INTO orders(order_id, user_id, total, created_date)
VALUES('4c841716-e28b-11e9-8960-00ff7b45daf6', '20004007533', 100.00, '2019-09-03 08:10:00');
SELECT @num;
```
![](https://yyc-images.oss-cn-beijing.aliyuncs.com/9_auto_increment_in_before_tigger_is_zero.png)

- 最后，触发器体内可以使用存储过程或函数么？

> 先说结论:  
> 1. 只允许以`out`参数形式返回数据的存储过程。  
> 2. 允许使用函数。  

<br>
``` sql
SHOW TRIGGERS in test;
DROP TRIGGER insert_get_auto_increment;

CREATE PROCEDURE uspGetOrderById(
    userId BIGINT,
    OUT rowId INT)
BEGIN
    SELECT id INTO rowId FROM orders
    WHERE user_id = userId;
END

CREATE TRIGGER insert_test_procedure_with_out_param BEFORE INSERT on orders
FOR EACH ROW
CALL uspGetOrderById(20004007596, @id);
SHOW TRIGGERS in test;

SET @id = 0;
INSERT INTO orders(order_id, user_id, total, created_date)
VALUES('4c841716-e28b-11e9-8960-00ff7b45daf6', '20004007533', 100.00, '2019-09-03 08:10:00');
SELECT @id;
```
![](https://yyc-images.oss-cn-beijing.aliyuncs.com/10_procedure_with_out_parameter_added_to_tigger_body.png)

<br>
``` sql
SHOW TRIGGERS in test;
DROP TRIGGER insert_test_procedure_with_out;

CREATE PROCEDURE uspGetTotalFromOrdersByOrderId(
    orderId VARCHAR(50))
BEGIN
    SELECT total FROM orders WHERE order_id = orderId;
END

CREATE TRIGGER insert_test_procedure_without_out_param BEFORE INSERT on orders
FOR EACH ROW
CALL uspGetTotalFromOrdersByOrderId('4c841716-e28b-11e9-8960-00ff7b45daf5');

INSERT INTO orders(order_id, user_id, total, created_date)
VALUES('4c841716-e28b-11e9-8960-00ff7b45daf6', '20004007533', 100.00, '2019-09-03 08:10:00');
```
![](https://yyc-images.oss-cn-beijing.aliyuncs.com/11_procedure_without_out_parameter_is_not_allowed_added_to_tigger_body.png)

<br>
``` sql
SHOW TRIGGERS in test;
DROP TRIGGER insert_test_procedure_without_out_param;

CREATE TRIGGER insert_test_function BEFORE INSERT on orders
FOR EACH ROW SET @str = Pascal('yyc');

SET @str = '';
INSERT INTO orders(order_id, user_id, total, created_date)
VALUES('4c841716-e28b-11e9-8960-00ff7b45daf6', '20004007533', 100.00, '2019-09-03 08:10:00');
SELECT @str;
```
![](https://yyc-images.oss-cn-beijing.aliyuncs.com/12_function_can_be_added_to_trigger_body.png)

<br>
## 如何查看所有触发器？  

``` sql
SHOW TRIGGERS
    [{FROM | IN} db_name]
    [LIKE 'pattern' | WHERE expr]
```

<br>
## 如何删除触发器？  

``` sql
DROP TRIGGER SCHEMA.TRIGGER_NAME;
```

> If you drop a table, any triggers for the table are also dropped.

由于触发器与表紧密关联，因此一旦删除某张表，该表涉及的触发器也会被删除。

``` sql
SHOW TRIGGERS in test;
DROP TABLE IF EXISTS orders;
SHOW TRIGGERS in test;
```
![](https://yyc-images.oss-cn-beijing.aliyuncs.com/13_1_has_insert_test_function.png)
<br>
![](https://yyc-images.oss-cn-beijing.aliyuncs.com/13_2_after_drop_orders_table.png)

<br>
## delimiter的作用？  

> 通过命令行方式操作`MySQL`时，一次可以执行多条数据。

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/14_delimiter.png)

<br>
## 参考  

- [24.3.1 Trigger Syntax and Examples](https://dev.mysql.com/doc/refman/8.0/en/trigger-syntax.html){:target="_blank"}
- [13.7.7.38 SHOW TRIGGERS Syntax](https://dev.mysql.com/doc/refman/8.0/en/show-triggers.html){:target="_blank"}

