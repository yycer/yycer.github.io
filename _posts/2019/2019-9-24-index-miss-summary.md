---
layout: post
title: MySQL索引未命中场景
category: database
tags: [database]
excerpt: 索引未命中整理
---
## 准备测试数据

- 创建一张表  

``` sql
create table person(
    id     int,
    name   varchar(20),
    gender char(6),
    email  varchar(50)
);
```

- 创建存储过程  

``` sql
delimiter $$ # 声明存储过程的结束符号为$$
create procedure insertPerson()
BEGIN
    declare i int default 1;
    while(i < 10000) do
        insert into person 
        values(i, concat('yyc',i),'male', concat('yyc',i,'@163.com'));
        set i = i + 1;
    end while;
END$$ # $$结束
delimiter; # 重新声明分号为结束符号

```

- 执行存储过程  

``` sql
call insertPerson();
```

<br>
## 索引相关基本操作

- 如何创建一个普通索引？
``` sql
CREATE INDEX index_name on table_name(column(length));
```

- 如何创建一个主键？
``` sql
ALTER TABLE table_name ADD PRIMARY KEY (column);
ALTER TABLE table_name ADD CONSTRAINT constraint_description PRIMARY KEY (column);
```

- 如何删除一个主键？
``` sql
ALTER TABLE table_name DROP PRIMARY KEY
```

- 如何创建一个唯一索引？
``` sql
CREATE UNIQUE INDEX index_name on table_name(column(length));
```

- 如何创建一个联合索引？
``` sql
CREATE INDEX index_name on table_name(column1, column2);
```

- 如何删除一个索引？
``` sql
DROP INDEX index_name on table_name;
```

- 如何显示表中已经创建的索引？
``` sql
SHOW INDEX FROM table_name;
```



<br>
## 模糊查询  

``` sql
# Index has been created for the name column.
EXPLAIN SELECT * FROM person WHERE name LIKE 'yy%333';
```
![模糊查询](https://yyc-images.oss-cn-beijing.aliyuncs.com/%E6%A8%A1%E7%B3%8A%E5%8C%B9%E9%85%8D.png?Expires=1569557355&OSSAccessKeyId=TMP.hVx9hnCuz9jD53vzWBzHJnGEEsCrHhc9qDSxBnz3qoqz4UWBh92zFtxwPj4MzvFTwA1XX8XGkUAZSbzTdosEEkACgCZUfzVHaMvMwNYso1eXc8eq5wwWurM45YCPmM.tmp&Signature=kGuG%2FphXwD2JE4j3p%2BZT5QqUIXw%3D)  

<br>
## 并集查询  

``` sql
# Index has been created for the name column.
EXPLAIN SELECT * FROM person WHERE `name` = 'yyc8888' OR id = 333;
```
![并集_idx_name](https://yyc-images.oss-cn-beijing.aliyuncs.com/%E5%B9%B6%E9%9B%86_idx_name.png?Expires=1569557379&OSSAccessKeyId=TMP.hVx9hnCuz9jD53vzWBzHJnGEEsCrHhc9qDSxBnz3qoqz4UWBh92zFtxwPj4MzvFTwA1XX8XGkUAZSbzTdosEEkACgCZUfzVHaMvMwNYso1eXc8eq5wwWurM45YCPmM.tmp&Signature=uz8pCnMvcw5TnoVycCwvuFYHntQ%3D)  
<br>
``` sql
# Index has been created for the name and id column.
EXPLAIN SELECT * FROM person WHERE `name` = 'yyc8888' OR id = 333;
```
![并集_idx_id_and_idx_name](https://yyc-images.oss-cn-beijing.aliyuncs.com/%E5%B9%B6%E9%9B%86_idx_id_and_idx_name.png?Expires=1569557391&OSSAccessKeyId=TMP.hVx9hnCuz9jD53vzWBzHJnGEEsCrHhc9qDSxBnz3qoqz4UWBh92zFtxwPj4MzvFTwA1XX8XGkUAZSbzTdosEEkACgCZUfzVHaMvMwNYso1eXc8eq5wwWurM45YCPmM.tmp&Signature=pSh%2BdWlMA49MB1lMDaUIhfd6rqg%3D)  


<br>
## 范围查询  

``` sql
# Index has been created for the id column.
EXPLAIN SELECT * FROM person WHERE id > 3000
```
![范围查询](https://yyc-images.oss-cn-beijing.aliyuncs.com/%E8%8C%83%E5%9B%B4%E6%9F%A5%E8%AF%A2.png?Expires=1569557407&OSSAccessKeyId=TMP.hVx9hnCuz9jD53vzWBzHJnGEEsCrHhc9qDSxBnz3qoqz4UWBh92zFtxwPj4MzvFTwA1XX8XGkUAZSbzTdosEEkACgCZUfzVHaMvMwNYso1eXc8eq5wwWurM45YCPmM.tmp&Signature=LMU7n6htGF8eboA67Hbnc9TSMjE%3D)  

<br>
## 使用函数  

``` sql
# Index has been created for the name column.
EXPLAIN SELECT * FROM person WHERE REVERSE(`name`) = '333cyy';
```
![使用函数](https://yyc-images.oss-cn-beijing.aliyuncs.com/%E4%BD%BF%E7%94%A8%E5%87%BD%E6%95%B0.png?Expires=1569557419&OSSAccessKeyId=TMP.hVx9hnCuz9jD53vzWBzHJnGEEsCrHhc9qDSxBnz3qoqz4UWBh92zFtxwPj4MzvFTwA1XX8XGkUAZSbzTdosEEkACgCZUfzVHaMvMwNYso1eXc8eq5wwWurM45YCPmM.tmp&Signature=ovRqAghlT7FxojuiLsfWlz1%2BP%2Fw%3D)  

<br>
## 存在类型转换  

``` sql
# Index has been created for the name column.
EXPLAIN SELECT * FROM person WHERE name = 8888
```
![存在类型转换](https://yyc-images.oss-cn-beijing.aliyuncs.com/%E5%AD%98%E5%9C%A8%E7%B1%BB%E5%9E%8B%E8%BD%AC%E6%8D%A2.png?Expires=1569557452&OSSAccessKeyId=TMP.hVx9hnCuz9jD53vzWBzHJnGEEsCrHhc9qDSxBnz3qoqz4UWBh92zFtxwPj4MzvFTwA1XX8XGkUAZSbzTdosEEkACgCZUfzVHaMvMwNYso1eXc8eq5wwWurM45YCPmM.tmp&Signature=QQfaE8s0nUvRxYypmQ%2BaexL78Ck%3D)  

<br>
## 联合索引不满足最左匹配原则   

``` sql
# Union index has been created for the name and id column.
EXPLAIN SELECT * FROM person WHERE id = 333 and `name` = 'yyc333';
```
![union_index_id_name](https://yyc-images.oss-cn-beijing.aliyuncs.com/%E8%81%94%E5%90%88%E7%B4%A2%E5%BC%95_id_name.png?Expires=1569557487&OSSAccessKeyId=TMP.hVx9hnCuz9jD53vzWBzHJnGEEsCrHhc9qDSxBnz3qoqz4UWBh92zFtxwPj4MzvFTwA1XX8XGkUAZSbzTdosEEkACgCZUfzVHaMvMwNYso1eXc8eq5wwWurM45YCPmM.tmp&Signature=JoYgWyGB06b%2F8osofA%2FSS9A6jBc%3D)  
<br>
``` sql
# Union index has been created for the name and id column.
EXPLAIN SELECT * FROM person WHERE id = 333;
```
![union_index_id](https://yyc-images.oss-cn-beijing.aliyuncs.com/%E8%81%94%E5%90%88%E7%B4%A2%E5%BC%95_id.png?Expires=1569557497&OSSAccessKeyId=TMP.hVx9hnCuz9jD53vzWBzHJnGEEsCrHhc9qDSxBnz3qoqz4UWBh92zFtxwPj4MzvFTwA1XX8XGkUAZSbzTdosEEkACgCZUfzVHaMvMwNYso1eXc8eq5wwWurM45YCPmM.tmp&Signature=G9y5n8f%2FI1Ime4XEOz4x9zoFrM8%3D)  
<br>
``` sql
# Union index has been created for the name and id column.
EXPLAIN SELECT * FROM person WHERE `name` = 'yyc333';
```
![union_index_name](https://yyc-images.oss-cn-beijing.aliyuncs.com/%E8%81%94%E5%90%88%E7%B4%A2%E5%BC%95_name.png?Expires=1569557506&OSSAccessKeyId=TMP.hVx9hnCuz9jD53vzWBzHJnGEEsCrHhc9qDSxBnz3qoqz4UWBh92zFtxwPj4MzvFTwA1XX8XGkUAZSbzTdosEEkACgCZUfzVHaMvMwNYso1eXc8eq5wwWurM45YCPmM.tmp&Signature=UvD3UVSaxr%2BwIemMlW1x5i2cuCE%3D)  

<br>
## 参考
[MySQL索引原理以及查询优化](https://www.cnblogs.com/bypp/p/7755307.html){:target="_blank"}