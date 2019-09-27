---
layout: post
title: 使用MySQL正则表达式与通配符
category: database
tags: [database]
excerpt: 整理MySQL正则与通配符用法
---
![](https://yyc-images.oss-cn-beijing.aliyuncs.com/MySQL_regular_expression.png?Expires=1569571774&OSSAccessKeyId=TMP.hVx9hnCuz9jD53vzWBzHJnGEEsCrHhc9qDSxBnz3qoqz4UWBh92zFtxwPj4MzvFTwA1XX8XGkUAZSbzTdosEEkACgCZUfzVHaMvMwNYso1eXc8eq5wwWurM45YCPmM.tmp&Signature=97pE%2FeFSTRt8hNBWQpjOvOT%2FJqo%3D)  

<br>
## 数据准备
``` sql  
CREATE TABLE IF NOT EXISTS products (
    id INT AUTO_INCREMENT PRIMARY KEY,
    prod_name VARCHAR(255) NOT NULL
);

INSERT INTO products(prod_name) VALUES('5X 100');
INSERT INTO products(prod_name) VALUES('5X 200');
INSERT INTO products(prod_name) VALUES('5X 300');
INSERT INTO products(prod_name) VALUES('3X 200');
INSERT INTO products(prod_name) VALUES('5.5X 300');
INSERT INTO products(prod_name) VALUES('8.5X 20');
INSERT INTO products(prod_name) VALUES('facewash');
```


<br>
## 基本字符匹配  

> 如何检索列`prod_name`包含文本100的所有行?  

``` sql  
SELECT * FROM products WHERE prod_name REGEXP '100'
```
![](https://yyc-images.oss-cn-beijing.aliyuncs.com/basic_usage.png?Expires=1569573032&OSSAccessKeyId=TMP.hVx9hnCuz9jD53vzWBzHJnGEEsCrHhc9qDSxBnz3qoqz4UWBh92zFtxwPj4MzvFTwA1XX8XGkUAZSbzTdosEEkACgCZUfzVHaMvMwNYso1eXc8eq5wwWurM45YCPmM.tmp&Signature=6sne3vKs5E5foid%2FTfLKQz4k6I8%3D)  

> `LIKE`与`REGEXP`的区别是什么？  

我们先来看一段SQL
``` sql  
SELECT * FROM products WHERE prod_name LIKE '100'
```
![](https://yyc-images.oss-cn-beijing.aliyuncs.com/like_100.png?Expires=1569573068&OSSAccessKeyId=TMP.hVx9hnCuz9jD53vzWBzHJnGEEsCrHhc9qDSxBnz3qoqz4UWBh92zFtxwPj4MzvFTwA1XX8XGkUAZSbzTdosEEkACgCZUfzVHaMvMwNYso1eXc8eq5wwWurM45YCPmM.tmp&Signature=z5U6YQzshWC4CqeXi9RrmrrmGwM%3D)  

它没有返回任何数据？为什么？

> 因为`LIKE`匹配的是`整个列`，如果被匹配的文本在`列值`中出现，`LIKE`操作符也不会找到它。而`REGEXP`在`列值`中进行匹配，所以会有相应的数据返回。如果我就想用`LIKE`返回列值中包含100的数据，应该怎么办？ 使用`通配符`。

``` sql  
SELECT * FROM products WHERE prod_name LIKE '%100';
SELECT * FROM products WHERE prod_name LIKE '_X 100';
```
<br>
![](https://yyc-images.oss-cn-beijing.aliyuncs.com/basic_usage.png?Expires=1569573032&OSSAccessKeyId=TMP.hVx9hnCuz9jD53vzWBzHJnGEEsCrHhc9qDSxBnz3qoqz4UWBh92zFtxwPj4MzvFTwA1XX8XGkUAZSbzTdosEEkACgCZUfzVHaMvMwNYso1eXc8eq5wwWurM45YCPmM.tmp&Signature=6sne3vKs5E5foid%2FTfLKQz4k6I8%3D)

<br>
## or匹配  

> 如何检索列`prod_name`包含文本100或200的所有行。

``` sql  
SELECT * FROM products WHERE prod_name REGEXP '100|200'
```
![](https://yyc-images.oss-cn-beijing.aliyuncs.com/mysql_regexp_or.png?Expires=1569573400&OSSAccessKeyId=TMP.hVx9hnCuz9jD53vzWBzHJnGEEsCrHhc9qDSxBnz3qoqz4UWBh92zFtxwPj4MzvFTwA1XX8XGkUAZSbzTdosEEkACgCZUfzVHaMvMwNYso1eXc8eq5wwWurM45YCPmM.tmp&Signature=r8e%2F%2FywHUStmOtsGXTcb4B%2BHsUU%3D)

<br>
## 匹配几个字符之一  

``` sql  
SELECT * FROM products WHERE prod_name REGEXP '5X [12]00'
```
![](https://yyc-images.oss-cn-beijing.aliyuncs.com/mysql_regexp_match.png?Expires=1569573417&OSSAccessKeyId=TMP.hVx9hnCuz9jD53vzWBzHJnGEEsCrHhc9qDSxBnz3qoqz4UWBh92zFtxwPj4MzvFTwA1XX8XGkUAZSbzTdosEEkACgCZUfzVHaMvMwNYso1eXc8eq5wwWurM45YCPmM.tmp&Signature=zZfXaQM2B2sQ44afNV7ySPDB7S0%3D)

<br>
## 匹配范围  

``` sql  
SELECT * FROM products WHERE prod_name REGEXP '5x [1-9]00'
```
![](https://yyc-images.oss-cn-beijing.aliyuncs.com/mysql_regexp_range.png?Expires=1569573429&OSSAccessKeyId=TMP.hVx9hnCuz9jD53vzWBzHJnGEEsCrHhc9qDSxBnz3qoqz4UWBh92zFtxwPj4MzvFTwA1XX8XGkUAZSbzTdosEEkACgCZUfzVHaMvMwNYso1eXc8eq5wwWurM45YCPmM.tmp&Signature=BtIprWUsLKwdPPC296dyg5tc1PU%3D)

<br>
## 匹配特殊字符  

``` sql  
SELECT * FROM products WHERE prod_name REGEXP '.'
```
> 很明显，这不是期望的结果。因为`.`匹配任意字符，而我们只想找出包含`.`字符的值，所以我们需要用到转义。 
![](https://yyc-images.oss-cn-beijing.aliyuncs.com/%E7%82%B9_%E5%B0%9A%E6%9C%AA%E8%BD%AC%E4%B9%89.png?Expires=1569573457&OSSAccessKeyId=TMP.hVx9hnCuz9jD53vzWBzHJnGEEsCrHhc9qDSxBnz3qoqz4UWBh92zFtxwPj4MzvFTwA1XX8XGkUAZSbzTdosEEkACgCZUfzVHaMvMwNYso1eXc8eq5wwWurM45YCPmM.tmp&Signature=EFQf0Akj7KVaiGs2OnaA8JiGmRg%3D)

``` sql  
SELECT * FROM products WHERE prod_name REGEXP '\\.'
```
![](https://yyc-images.oss-cn-beijing.aliyuncs.com/%E7%82%B9_%E8%BD%AC%E4%B9%89.png?Expires=1569573473&OSSAccessKeyId=TMP.hVx9hnCuz9jD53vzWBzHJnGEEsCrHhc9qDSxBnz3qoqz4UWBh92zFtxwPj4MzvFTwA1XX8XGkUAZSbzTdosEEkACgCZUfzVHaMvMwNYso1eXc8eq5wwWurM45YCPmM.tmp&Signature=d4jqTSG5V0zfQ%2FKHpnzdk0jkTWA%3D)

<br>
## 匹配字符类  

``` sql  
SELECT * FROM products WHERE prod_name REGEXP '\\.[:digit:][:alpha:]'
```
![](https://yyc-images.oss-cn-beijing.aliyuncs.com/mysql_regexp_charset.png?Expires=1569573487&OSSAccessKeyId=TMP.hVx9hnCuz9jD53vzWBzHJnGEEsCrHhc9qDSxBnz3qoqz4UWBh92zFtxwPj4MzvFTwA1XX8XGkUAZSbzTdosEEkACgCZUfzVHaMvMwNYso1eXc8eq5wwWurM45YCPmM.tmp&Signature=KDmKGfuuEad8KCCDMnlSCO5hdHU%3D)

<br>
## 匹配多个实例  

> 检索包含8个字母的值。  


``` sql  
SELECT * FROM products WHERE prod_name REGEXP '[:alpha:]{8}'
```
![](https://yyc-images.oss-cn-beijing.aliyuncs.com/mysql_regexp_match_multi.png?Expires=1569573499&OSSAccessKeyId=TMP.hVx9hnCuz9jD53vzWBzHJnGEEsCrHhc9qDSxBnz3qoqz4UWBh92zFtxwPj4MzvFTwA1XX8XGkUAZSbzTdosEEkACgCZUfzVHaMvMwNYso1eXc8eq5wwWurM45YCPmM.tmp&Signature=8AbYqVqIjBZlK4Ivv7vSSDCby6A%3D)

<br>
## 定位符  

> 匹配以8开头，0结尾的值。  

``` sql  
SELECT * FROM products WHERE prod_name REGEXP '^8.*0$'
```
![](https://yyc-images.oss-cn-beijing.aliyuncs.com/mysql_regexp_locate.png?Expires=1569573513&OSSAccessKeyId=TMP.hVx9hnCuz9jD53vzWBzHJnGEEsCrHhc9qDSxBnz3qoqz4UWBh92zFtxwPj4MzvFTwA1XX8XGkUAZSbzTdosEEkACgCZUfzVHaMvMwNYso1eXc8eq5wwWurM45YCPmM.tmp&Signature=DENMQCNJj%2Fwtzfc8elYK%2BjdRP%2F4%3D)

## 参考
- 《MySQL必知必会》 第八章 - 用通配符进行过滤
- 《MySQL必知必会》 第九章 - 用正则表达式进行搜索

