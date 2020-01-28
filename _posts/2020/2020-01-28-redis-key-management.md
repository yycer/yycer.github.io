---
layout: post
title: Redis - 键管理
category: redis
tags: [redis]
excerpt: Redis Key Management
---

## 管理单个键  

#### 键重命名  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/rename.png)  

``` shell
127.0.0.1:6379> set fruit apple
OK
127.0.0.1:6379> get fruit
"apple"
127.0.0.1:6379> rename fruit phone
OK
127.0.0.1:6379> get fruit // old key name
(nil)
127.0.0.1:6379> get phone // new key name
"apple"
---------------------------------------------
127.0.0.1:6379> del fruit phone
(integer) 1
127.0.0.1:6379> set fruit apple
OK
127.0.0.1:6379> get fruit
"apple"
127.0.0.1:6379> rename fruit phone
OK
127.0.0.1:6379> get fruit // old key name
(nil)
127.0.0.1:6379> get phone // new key name
"apple"
```

#### 随机返回一个键  

``` shell
127.0.0.1:6379> randomkey
"user:ranking:2"
127.0.0.1:6379> randomkey
"user:ranking:2"
127.0.0.1:6379> randomkey
"sorted_set_inter_test"
```

#### 键过期  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/expire_methods.png)  

<br>
![](https://yyc-images.oss-cn-beijing.aliyuncs.com/ttl.png)  


``` shell
127.0.0.1:6379> set name yyc
OK
127.0.0.1:6379> expire name 10
(integer) 1
127.0.0.1:6379> ttl name
(integer) 7
127.0.0.1:6379> ttl name
(integer) 1
127.0.0.1:6379> ttl name
(integer) -2
127.0.0.1:6379> get name
(nil)
---------------------------------------------
127.0.0.1:6379> get name
"yyc"
127.0.0.1:6379> expireat name 1580170110
(integer) 1
127.0.0.1:6379> ttl name
(integer) 12
127.0.0.1:6379> ttl name
(integer) 9
127.0.0.1:6379> get name
"yyc"
127.0.0.1:6379> ttl name
(integer) 4
127.0.0.1:6379> get name
(nil)
```

有一点需要注意：对于字符串类型键，如果执行set命令，会去掉过期时间！  

``` shell
127.0.0.1:6379> set name yyc
OK
127.0.0.1:6379> pexpire name 10000
(integer) 1
127.0.0.1:6379> ttl name
(integer) 8
127.0.0.1:6379> set name frankie
OK
127.0.0.1:6379> ttl name
(integer) -1
```

#### 键迁移  

准备工作：  

``` shell
redis-server.exe --port 6379
redis-server.exe --port 6380
redis-cli.exe -p 6379
redis-cli.exe -p 6380
```

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/env_simulate.png)  

<br>
![](https://yyc-images.oss-cn-beijing.aliyuncs.com/migrate.png)  


第一种情况：  

``` shell
127.0.0.1:6379> get name
(nil)
127.0.0.1:6379> set name yyc
OK
127.0.0.1:6379> get name
"yyc"
127.0.0.1:6379> migrate 127.0.0.1 6380 name 0 1000 copy
OK
127.0.0.1:6379> get name
"yyc"
---------------------------------------------
127.0.0.1:6380> get name
"yyc"
```

第二种情况：  
``` shell
127.0.0.1:6380> set name frankie
OK
127.0.0.1:6380> get name
"frankie"
127.0.0.1:6379> migrate 127.0.0.1 6380 name 0 1000 copy replace
OK
127.0.0.1:6379> get name
"yyc"
---------------------------------------------
127.0.0.1:6380> get name
"yyc"
```

第三种情况：  
``` shell
127.0.0.1:6379> get name
(nil)
127.0.0.1:6379> migrate 127.0.0.1 6380 name 0 1000 copy
NOKEY
```

## 遍历键  

推荐使用渐进式遍历：  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/scan.png)  

``` shell
127.0.0.1:6379> scan 0
1) "34"
2)  1) "set_test"
    2) "num39"
    3) "num4"
    4) "user:2"
    5) "test_list_max_ziplist_entries"
    6) "num3"
    7) "sorted_set_test2"
    8) "user:ranking:1"
    9) "num40"
   10) "sorted_set_inter_test_sum"
   11) "address"

127.0.0.1:6379> scan 0 match num*
1) "34"
2) 1) "num39"
   2) "num4"
   3) "num3"
   4) "num40"

127.0.0.1:6379> scan 0 match num* count 2
1) "40"
2) 1) "num39"
127.0.0.1:6379> scan 0 match num* count 3
1) "24"
2) 1) "num39"
   2) "num4"
```

## 数据库管理  

- 如何知道当前使用的是哪个数据库？  

``` shell
127.0.0.1:6379> info keyspace
# Keyspace
db0:keys=34,expires=0,avg_ttl=0
```

- 如何选择数据库？  

``` shell
select index
127.0.0.1:6379[index]>...
```

- 如何知道当前数据库中键总个数？  

``` shell
127.0.0.1:6379> dbsize
(integer) 34
```

- 如何清空当前数据库所有键？  

``` shell
127.0.0.1:6379> dbsize
(integer) 34
127.0.0.1:6379> flushdb
OK
127.0.0.1:6379> dbsize
(integer) 0
```

- 如何清空所有数据库所有键？  

``` shell
127.0.0.1:6379> select 5
OK
127.0.0.1:6379[5]> dbsize
(integer) 1
127.0.0.1:6379[5]> select 0
OK
127.0.0.1:6379> set name frankie
OK
127.0.0.1:6379> dbsize
(integer) 1
127.0.0.1:6379> flushall
OK
127.0.0.1:6379> dbsize
(integer) 0
127.0.0.1:6379> select 5
OK
127.0.0.1:6379[5]> dbsize
(integer) 0
```

## `Reference`  
- `《Redis开发与运维》 - 2.7 键管理`