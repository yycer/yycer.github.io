---
layout: post
title: MySQL常用函数
category: database
tags: [database]
excerpt: 梳理MySQL常用函数
---
![](https://yyc-images.oss-cn-beijing.aliyuncs.com/MySQL_regular_expression.png?Expires=1569571774&OSSAccessKeyId=TMP.hVx9hnCuz9jD53vzWBzHJnGEEsCrHhc9qDSxBnz3qoqz4UWBh92zFtxwPj4MzvFTwA1XX8XGkUAZSbzTdosEEkACgCZUfzVHaMvMwNYso1eXc8eq5wwWurM45YCPmM.tmp&Signature=97pE%2FeFSTRt8hNBWQpjOvOT%2FJqo%3D)  

## 数据准备  

``` sql
CREATE TABLE IF NOT EXISTS orders (
    id           INT     AUTO_INCREMENT PRIMARY KEY,
    user_id      BIGINT  NOT NULL,
    total        DECIMAL NOT NULL,
    created_date DATETIME 
);

INSERT INTO orders(user_id, total, created_date) VALUES('123', 20.00, '2019-9-20 10:10:30');
INSERT INTO orders(user_id, total, created_date) VALUES('382', 30.00, '2019-8-13 10:10:30');
INSERT INTO orders(user_id, total, created_date) VALUES('829', 80.00, '2019-9-30 10:10:30');

```


<br>
## 文本处理  

- 如何去除字符串中多余的空格？  

``` sql
SET @str = '  123   ';
SELECT LENGTH(@str)        as all_length;           # 8
SELECT LENGTH(TRIM(@str))  as word_length;        # 3
SELECT LENGTH(LTRIM(@str)) as remove_left_length;   # 6
SELECT LENGTH(RTRIM(@str)) as remove_right_length; # 5
```

- 如何将字符串全部转换为大小写？

``` sql
SET @str = 'Yyc';
SELECT UPPER(@str) as upper; # YYC
SELECT LOWER(@str) as lower; # yyc
```

<br>
## 日期处理  

- 如何获得当前时间？  

``` sql
SELECT NOW()      # 2019-09-27 17:38:08
SELECT CURDATE()  # 2019-09-27
SELECT CURTIME()  # 17:38:08
```


- 如何获得2019年9月份的所有订单记录？  

> 第一种方式: `BETWEEN...AND...`  

  
  ``` sql
  SELECT * FROM orders 
  WHERE Date(created_date) BETWEEN '2019-09-01' AND '2019-09-30'
  ```

> 第二种方式: `YEAR()`和`MONTH()`  

  
  ``` sql
  SELECT * FROM orders 
  WHERE YEAR(created_date) = 2019 AND MONTH(created_date) = 9
  ```

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/datetime_processed.png?Expires=1569580945&OSSAccessKeyId=TMP.hVx9hnCuz9jD53vzWBzHJnGEEsCrHhc9qDSxBnz3qoqz4UWBh92zFtxwPj4MzvFTwA1XX8XGkUAZSbzTdosEEkACgCZUfzVHaMvMwNYso1eXc8eq5wwWurM45YCPmM.tmp&Signature=eVZ9S%2F66oNln1n2BNVazx5tKa1s%3D)


<br>
## 参考  
- 《MySQL必知必会》 第十一章 - 使用数据处理函数
