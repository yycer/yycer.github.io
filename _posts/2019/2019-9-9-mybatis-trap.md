---
layout: post
title: MyBatis & SQL Server遇到的坑(类型转换)
category: database
tags: [mybatis, sql-server, database]
excerpt: 记录MyBatis的坑
---
## MyBatis SQL语句遇到的性能问题
> 1.场景还原  

&emsp;&emsp;首先，我们使用SQL Server数据库，创建一张`User`表，其中包含userId、userName、gender字段，其中userId的数据类型为`char(20)`，此时我们想通过`userId`获得这个人的姓名。

&emsp;&emsp;这段SQL很简单： ```SELECT userName FROM dbo.User (nolock) WHERE userId = '100'```

> 2.问题描述  

&emsp;&emsp;上面这段简单的SQL语句却隐藏着很一个严重的性能问题：当MyBatis生成该语句，并在SQL Server执行时，参数userId的JDBC Type为`nvarchar(4000)`，但表中userId的数据类型为`char(20)`，因此必然存在着`类型转换`。

&emsp;&emsp;在压力测试场景，或调用频繁的情况下，导致SQL Server CPU严重负荷，以及服务吞吐量严重下降。  

<br>
## char、varchar、nvarchar区别
- char：对于英文字母占1个字节，对于汉字占2个字节，char属于`定长类型`数据结构，剩余空间全部使用空格填补，因此索引效率极高。
- varchar： 多余空间不会使用空格填补，实际长度为字符串长度+1，这个1代表字符串的长度。
- nvarchar：所有的字符都占用2个字节，无论英文字母，还是汉字。解决了多字符集之间的转换问题，N代表Unicode。  

<br>
## 参考
- [nchar，char，varchar与nvarchar区别](https://www.cnblogs.com/lichang1987/archive/2009/03/04/1403166.html)