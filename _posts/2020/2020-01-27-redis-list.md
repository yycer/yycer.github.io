---
layout: post
title: Redis - 列表
category: redis
tags: [redis]
excerpt: Redis List
---

## 特点  

- 有序  
- 可重复  

## 常用命令  

- 增  

``` shell
# 1. rpush key value [value ...]
127.0.0.1:6379> rpush list_insert 1 2 3
(integer) 3
# 2. lpush key value [value ...]
127.0.0.1:6379> lpush list_insert 4 5 6
(integer) 6
127.0.0.1:6379> lrange list_insert 0 -1
1) "6"
2) "5"
3) "4"
4) "1"
5) "2"
6) "3"
# 3. linsert key before|after pivot value
127.0.0.1:6379> linsert list_insert before 1 yyc
(integer) 7
127.0.0.1:6379> lrange list_insert 0 -1
1) "6"
2) "5"
3) "4"
4) "yyc"
5) "1"
6) "2"
7) "3"
```


- 删  

``` shell
127.0.0.1:6379> lrange list_insert 0 -1
1) "6"
2) "5"
3) "4"
4) "yyc"
5) "1"
6) "2"
7) "3"
# 1. lpop key
127.0.0.1:6379> lpop list_insert
"6"
# 2. rpop key
127.0.0.1:6379> rpop list_insert
"3"
127.0.0.1:6379> lrange list_insert 0 -1
1) "5"
2) "4"
3) "yyc"
4) "1"
5) "2"
# 3. ltrim key start end
127.0.0.1:6379> lrange list_insert 1 3
1) "4"
2) "yyc"
3) "1"
```

重点说下，如何删除指定元素： `lrem key count value`  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/lrem.png)  

证明：  

``` shell
127.0.0.1:6379> rpush list_lrem 1 2 3 1 2 1 3 1
(integer) 8
127.0.0.1:6379> lrange list_lrem 0 -1
1) "1"
2) "2"
3) "3"
4) "1"
5) "2"
6) "1"
7) "3"
8) "1"
# 1. count > 0
127.0.0.1:6379> lrem list_lrem 3 1
(integer) 3
127.0.0.1:6379> lrange list_lrem 0 -1
1) "2"
2) "3"
3) "2"
4) "3"
5) "1"

127.0.0.1:6379> del list_lrem
(integer) 1
127.0.0.1:6379> rpush list_lrem 1 2 3 1 2 1 3 1
(integer) 8
# 2. count < 0
127.0.0.1:6379> lrem list_lrem -3 1
(integer) 3
127.0.0.1:6379> lrange list_lrem 0 -1
1) "1"
2) "2"
3) "3"
4) "2"
5) "3"

127.0.0.1:6379> del list_lrem
(integer) 1
127.0.0.1:6379> rpush list_lrem 1 2 3 1 2 1 3 1
(integer) 8
# 3. count = 0
127.0.0.1:6379> lrem list_lrem 0 1
(integer) 4
127.0.0.1:6379> lrange list_lrem 0 -1
1) "2"
2) "3"
3) "2"
4) "3"
```

- 改  

``` shell
127.0.0.1:6379> lrange list_lrem 0 -1
1) "2"
2) "3"
3) "2"
4) "3"
# 1. lset key index newValue
127.0.0.1:6379> lset list_lrem 2 8
OK
127.0.0.1:6379> lrange list_lrem 0 -1
1) "2"
2) "3"
3) "8"
4) "3"
```


- 查  

先来单独讲下`lrange`方法：  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/lrange.png)  


``` shell
127.0.0.1:6379> lrange list_query 0 -1
 1) "0"
 2) "1"
 3) "2"
 4) "3"
 5) "4"
 6) "5"
 7) "6"
 8) "7"
 9) "8"
10) "9"
127.0.0.1:6379> lrange list_query -3 -8
(empty list or set)
127.0.0.1:6379> lrange list_query -8 -3
1) "2"
2) "3"
3) "4"
4) "5"
5) "6"
6) "7"
127.0.0.1:6379> lrange list_query 2 7
1) "2"
2) "3"
3) "4"
4) "5"
5) "6"
6) "7"
```

再提下如何根据索引查找、获得列表长度：  

``` shell
127.0.0.1:6379> lrange list_query 0 -1
 1) "0"
 2) "1"
 3) "2"
 4) "3"
 5) "4"
 6) "5"
 7) "6"
 8) "7"
 9) "8"
10) "9"
# 1. lindex key index
127.0.0.1:6379> lindex list_query 4
"4"
# 2. llen key
127.0.0.1:6379> llen list_query
(integer) 10
```

- 阻塞  

``` shell
blpop key [key ...] timeout
brpop key [key ...] timeout
```

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/brpop.png)  




## 内部编码  

列表的内部编码主要有如下两种：  

- `ziplist`  
- `linkedlist`  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/list_encoding.png)  


证明：  

``` shell
# Redis 3.0.503 (00000000/0) 64 bit

127.0.0.1:6379> llen test_list_max_ziplist_entries
(integer) 512
127.0.0.1:6379> object encoding test_list_max_ziplist_entries
"ziplist"
127.0.0.1:6379> rpush test_list_max_ziplist_entries 513
(integer) 513
127.0.0.1:6379> object encoding test_list_max_ziplist_entries
"linkedlist"
```


## 使用场景  

- `阻塞队列`  
- 文章列表  


## `Reference`  
- `《Redis开发与运维》 - 2.4 列表`  