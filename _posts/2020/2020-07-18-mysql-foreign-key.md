---
layout: post
title: MySQL - 外键
category: database
tags: [database]
excerpt: MySQL Foreign Key
---


我们先创建两张表，父表`t_class`，子表`t_student`：  

其中，父表包含班级编号`cid`、班级名称`cname`两个字段，且`cid`为主键。  

子表包含学生编号`sid`、学生姓名`sname`、班级编号`cid`三个字段，  

其中`sid`为主键，`cid`为外键。  


```
CREATE TABLE IF NOT EXISTS t_class(
    cid   INT(10)     AUTO_INCREMENT,
    cname VARCHAR(50) NOT NULL,
    PRIMARY KEY(cid)
);

CREATE TABLE IF NOT EXISTS t_student(
    sid    INT(10)     AUTO_INCREMENT,
    sname  VARCHAR(50) NOT NULL,
    cid    INT(10),
    PRIMARY KEY(sid),
    FOREIGN KEY(cid) REFERENCES t_class(cid)
);


INSERT INTO t_class(cname) VALUES('高一(1)班');
INSERT INTO t_class(cname) VALUES('高一(2)班');
INSERT INTO t_class(cname) VALUES('高一(3)班');

INSERT INTO t_student(sname, cid) VALUES('张三', 2);
INSERT INTO t_student(sname, cid) VALUES('李四', 1);
INSERT INTO t_student(sname, cid) VALUES('王五', 3);
```


主要注意以下三点：  


第一，外键字段去引用一张表中的某个字段时，该字段必须具有`唯一性`。  


也就说，父表中的`cid`字段一定要保证唯一性。  


第二，外键值可以为`null`。  

```
INSERT INTO t_student(sname, cid) VALUES('yyc', null);
```

举个例子：  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/value_is_null.png)  

第三，父子表的创建、插入、删除顺序。  

假设父表是`t_class`，子表是`t_student`。  

- 先创父表，再创子表。  

也就是先创建`t_class`表，然后创建`t_student`表，因为后者的`cid`字段依赖于前者。  


- 先插入父表，再插入子表。  

往`t_student`插入一条`cid`为非空的数据之前，必须要将对应`cid`数据插入到`t_class`表中。  


- 先删除子表中的相关数据，再删除父表中的数据。  

举个例子：  

``` 
-- Cannot delete or update a parent row: 
-- a foreign key constraint fails (`test`.`t_student`, CONSTRAINT `t_student_ibfk_1` 
-- FOREIGN KEY (`cid`) REFERENCES `t_class` (`cid`))
DELETE FROM t_class WHERE cid = 2;
```
