---
layout: post
title: Redis - 哈希
category: redis
tags: [redis]
excerpt: Redis Hash
---
## 常用命令  

首先说明下`key`、`field`、`value`之间的关系：  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/hash_base_structure.png)

常用命令可以参考： [redis常用命令整理](http://yaoyichen.cn/redis/2019/09/22/redis_useful_commands.html){:target="_blank"}



## 内部编码  

哈希类型共有两种内部编码：  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/hash_encoding.png)

证明： 

``` shell
# Redis 3.0.503 (00000000/0) 64 bit

127.0.0.1:6379> hset user:2 name frankie
(integer) 1
127.0.0.1:6379> object encoding user:2
"ziplist"
127.0.0.1:6379> hset user:2 info1 1111111111222222222233333333334444444444555555555566666666667777
(integer) 1
127.0.0.1:6379> object encoding user:2
"ziplist"
127.0.0.1:6379> hset user:2 info2 11111111112222222222333333333344444444445555555555666666666677777
(integer) 1
127.0.0.1:6379> object encoding user:2
"hashtable"

127.0.0.1:6379> hlen test_hash_max_ziplist_entries
(integer) 512
127.0.0.1:6379> object encoding test_hash_max_ziplist_entries
"ziplist"
127.0.0.1:6379> hmset test_hash_max_ziplist_entries f513 v513
OK
127.0.0.1:6379> object encoding test_hash_max_ziplist_entries
"hashtable"
```

## 使用场景  

- 缓存  
- 共享`Session`  


## `Reference`  

- `《Redis开发与运维》 - 2.3 哈希`  