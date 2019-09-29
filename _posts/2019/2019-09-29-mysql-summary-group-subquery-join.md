---
layout: post
title: MySQL - 汇总、分组、子查询、联结用法
category: database
tags: [database]
excerpt: summary、group、subquery、join
---
![](https://yyc-images.oss-cn-beijing.aliyuncs.com/summary-group-subquery-union.png)

## 数据准备  

``` sql
CREATE TABLE IF NOT EXISTS orders(
    order_id VARCHAR(30),
    user_id  VARCHAR(20),
    total    DECIMAL not null,
    created_date datetime
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

CREATE TABLE IF NOT EXISTS users(
    user_id   VARCHAR(20),
    user_name VARCHAR(20)
);

INSERT INTO users(user_id, user_name) VALUES('20004007596', '张三');
INSERT INTO users(user_id, user_name) VALUES('20004007588', '李四');
INSERT INTO users(user_id, user_name) VALUES('20004007539', '王五');
INSERT INTO users(user_id, user_name) VALUES('20004007542', '胖子');
INSERT INTO users(user_id, user_name) VALUES('20004007501', '老王');

CREATE TABLE IF NOT EXISTS order_items(
    order_id VARCHAR(30),
    prod_id  VARCHAR(10)
);

INSERT INTO order_items(order_id, prod_id) VALUES('4c841716-e28b-11e9-8960-00ff7b45daf1', '82301021');
INSERT INTO order_items(order_id, prod_id) VALUES('4c841716-e28b-11e9-8960-00ff7b45daf1', '82301022');
INSERT INTO order_items(order_id, prod_id) VALUES('4c841716-e28b-11e9-8960-00ff7b45daf1', '82301023');
INSERT INTO order_items(order_id, prod_id) VALUES('4c841716-e28b-11e9-8960-00ff7b45daf1', '82301024');
INSERT INTO order_items(order_id, prod_id) VALUES('4c841716-e28b-11e9-8960-00ff7b45daf2', '82301021');
INSERT INTO order_items(order_id, prod_id) VALUES('4c841716-e28b-11e9-8960-00ff7b45daf2', '82301023');
INSERT INTO order_items(order_id, prod_id) VALUES('4c841716-e28b-11e9-8960-00ff7b45daf3', '82301025');
INSERT INTO order_items(order_id, prod_id) VALUES('4c841716-e28b-11e9-8960-00ff7b45daf4', '82301022');
INSERT INTO order_items(order_id, prod_id) VALUES('4c841716-e28b-11e9-8960-00ff7b45daf5', '82301023');
INSERT INTO order_items(order_id, prod_id) VALUES('4c841716-e28b-11e9-8960-00ff7b45daf5', '82301029');

CREATE TABLE IF NOT EXISTS products(
    prod_id    VARCHAR(10),
    prod_name  VARCHAR(100),
    prod_price DECIMAL,
    vendor_id  int
);

INSERT INTO products(prod_id, prod_name, prod_price, vendor_id) VALUES('82301021', '百雀羚洗面奶', 20.00, 1);
INSERT INTO products(prod_id, prod_name, prod_price, vendor_id) VALUES('82301022', '百雀羚面霜', 30.00, 1);
INSERT INTO products(prod_id, prod_name, prod_price, vendor_id) VALUES('82301029', '百雀眼部精华液', 40.00, 1);
INSERT INTO products(prod_id, prod_name, prod_price, vendor_id) VALUES('82301025', '相宜本草面膜', 25.00, 2);
INSERT INTO products(prod_id, prod_name, prod_price, vendor_id) VALUES('82301021', '欧莱雅日霜', 30.00, 3);
INSERT INTO products(prod_id, prod_name, prod_price, vendor_id) VALUES('82301022', '欧莱雅晚霜', 40.00, 3);
INSERT INTO products(prod_id, prod_name, prod_price, vendor_id) VALUES('82301023', '欧莱雅面膜', 90.00, 3);
INSERT INTO products(prod_id, prod_name, prod_price, vendor_id) VALUES('82301024', '欧莱雅洗面奶', 50.00, 3);


CREATE TABLE IF NOT EXISTS vendors(
    vendor_id   int,
    vendor_name VARCHAR(100)
);

INSERT INTO vendors(vendor_id, vendor_name) VALUES(1, '百雀羚');
INSERT INTO vendors(vendor_id, vendor_name) VALUES(2, '相宜本草');
INSERT INTO vendors(vendor_id, vendor_name) VALUES(3, '欧莱雅');
```

<br>
## 汇总数据  

> 如何获得vendor_id = 1的商品总数量？  

``` sql
SELECT count(prod_id) as prod_count FROM products where vendor_id = 1; # 3
```

> 如何获得vendor_id = 1的商品总价？

``` sql
SELECT SUM(prod_price) as sum_price FROM products where vendor_id = 1; # 90
```

> 如何获得vendor_id = 1的商品平均价格？  

``` sql
SELECT AVG(prod_price) as avg_price FROM products where vendor_id = 1; # 30
```

> 如何获得vendor_id = 1中最贵商品价格？

``` sql
SELECT MAX(prod_price) as max_price FROM products where vendor_id = 1; # 40
```

> 如何获得vendor_id = 1中最便宜商品价格？  

``` sql
SELECT MIN(prod_price) as min_price FROM products where vendor_id = 1; # 20
```


<br>
## 分组数据  

> 如何获得每位供应商提供的商品总数量？  

``` sql
SELECT vendor_id, count(prod_id) as prod_count FROM products
GROUP BY vendor_id;
```
![](https://yyc-images.oss-cn-beijing.aliyuncs.com/count_products_by_each_vendor.png)

> 如何筛选出提供2个以上商品的供应商？

``` sql
SELECT vendor_id, count(prod_id) as prod_count FROM products
GROUP BY vendor_id
HAVING count(prod_id) > 2;
```
![](https://yyc-images.oss-cn-beijing.aliyuncs.com/group_by_greater_than_2.png)

<br>
## 子查询  

> 如何获得购买了prod_id = 82301021的用户名称？

``` sql
# Step1: Get order_id list from order_items by prod_id.
SELECT order_id FROM order_items WHERE prod_id = '82301021';

# Step2: Get user_id list from orders by order_id.
SELECT user_id FROM orders
WHERE order_id REGEXP '4c841716-e28b-11e9-8960-00ff7b45daf1|4c841716-e28b-11e9-8960-00ff7b45daf2';

# Step3: Get user info list from users by user_id.
SELECT user_id, user_name FROM users WHERE user_id in ('20004007596', '20004007588');

```

> 汇总成一条语句。  

``` sql
SELECT user_id, user_name FROM users
WHERE user_id in (
                  SELECT user_id FROM orders
                  WHERE order_id in (
                                     SELECT order_id FROM order_items
                                     WHERE prod_id = '82301021'));
```

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/subquery-result.png)

<br>
## 联结表  

> 使用联结表的方式，完成上述子查询功能。  

``` sql
SELECT o.order_id, u.user_id, u.user_name
from orders as o
inner join order_items as items
on o.order_id = items.order_id
inner join users as u
on o.user_id = u.user_id
WHERE items.prod_id = '82301021';

```
![](https://yyc-images.oss-cn-beijing.aliyuncs.com/inner-join-result.png)

<br>
## 参考  

- 《MySQL必知必会》 第十二章 - 汇总数据
- 《MySQL必知必会》 第十三章 - 分组数据
- 《MySQL必知必会》 第十四章 - 使用子查询
- 《MySQL必知必会》 第十五章 - 联结表
- 《MySQL必知必会》 第十六章 - 创建高级联结



