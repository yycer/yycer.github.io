---
layout: post
title: Redis - String
category: redis
tags: [redis]
excerpt: Redis String
---

## 全局命令  

``` shell
# Get all keys, should not use in the normal environment.
127.0.0.1:6379> keys *
1) "name"
2) "book"
3) "num4"
4) "num3"
5) "num2"
6) "name39"
7) "num"
8) "address"
9) "name40"

# Get the numbers of keys.
127.0.0.1:6379> dbsize
(integer) 9

# Determine key is existed.
127.0.0.1:6379> exists num
(integer) 1

# Delete key.
127.0.0.1:6379> del num
(integer) 1
127.0.0.1:6379> exists num
(integer) 0

# Set expire time[seconds] to key.
127.0.0.1:6379> expire num2 100
(integer) 1

# Get the resting expire time.
127.0.0.1:6379> ttl num2
(integer) 97

# Get the type of key.
127.0.0.1:6379> type num2
string
127.0.0.1:6379> get num2
"1"
```


## 常用命令  


- 设置并获取值  

> set key value [ex seconds] [px milliseconds] [nx|xx]  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/set_properties.png)



``` shell
# set key value
127.0.0.1:6379> set address shanghai    
OK                                      
127.0.0.1:6379> get address             
"shanghai"     

# ex seconds                         
127.0.0.1:6379> set book java ex 120    
OK                                      
127.0.0.1:6379> get book                
"java"                          
# time to live        
127.0.0.1:6379> ttl book                
(integer) 110                           

# nx                               
127.0.0.1:6379> set book redis nx       
(nil)                                   
# xx
127.0.0.1:6379> set book redis xx       
OK                                      
127.0.0.1:6379> get book                
"redis"

# If you try to get not exists key, it will return nil.
127.0.0.1:6379> get not_exists_key
(nil)                                 
```

- 批量设置并获取值  

首先需要明确下为什么要进行批量处理？  

主要是为了`节省网络传输时间`。  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/why_use_multiply_operations.png)


``` shell
127.0.0.1:6379> mset a 1 b 2 c 3 d 4
OK

127.0.0.1:6379> mget a b c d
1) "1"
2) "2"
3) "3"
4) "4"
```

- 计数  

分为如下三种情况：  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/incr_logic.png)

``` shell
127.0.0.1:6379> exists num2
(integer) 0
127.0.0.1:6379> incr num2
(integer) 1

127.0.0.1:6379> set num yyc
OK
127.0.0.1:6379> get num
"yyc"
127.0.0.1:6379> incr num
(error) ERR value is not an integer or out of range

127.0.0.1:6379> set num3 10
OK
127.0.0.1:6379> incr num3
(integer) 11
```

除了`incr`命令，`Redis`还提供了`decr自减`、`incrby自增指定数字`、`decrby自减指定数字`命令。  

``` shell
127.0.0.1:6379> set num4 10
OK
127.0.0.1:6379> incrby num4 5
(integer) 15
127.0.0.1:6379> decr num4
(integer) 14
127.0.0.1:6379> decrby num4 4
(integer) 10
```


## 内部编码  

首先来看下字符串类型的三种内部编码方式：  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/string_internal_encoding.png)

``` shell

127.0.0.1:6379> info
# Server
redis_version:3.0.503
...

127.0.0.1:6379> set num39 111111111122222222223333333333444444444
OK
127.0.0.1:6379> object encoding num39
"embstr"
127.0.0.1:6379> set num40 1111111111222222222233333333334444444444
OK
127.0.0.1:6379> object encoding num40
"raw"
```

``` shell

127.0.0.1:6379> info
# Server
redis_version:3.2.6
...

127.0.0.1:6379> set num44 11111111112222222222333333333344444444445555
OK
127.0.0.1:6379> object encoding num44
"embstr"
127.0.0.1:6379> set num45 1111111111222222222233333333334444444444555555
OK
127.0.0.1:6379> object encoding num45
"raw"

```

我们以`3.2`之后的版本为例，解释下为什么以`44`个字节为分界线。  

首先我们来了解下Redis对象头结构：  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/redisObject.png)

`0.5 * 2 + 3 + 4 + 8 = 16 Bytes`，因此一个RedisObject对象头结构需要占据`16`字节的存储空间。  

再来看下`SDS[Simple Dynamic String]`结构体的大小：  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/simple_dynamic_string.png)

结合前面两点，可知分配一个字符串的最小占用空间为`19(16+3)`个字节。  

接下来的问题是：当字符串占用多少空间时，`Redis`才认为它是一个大字符串？  

答案是：`超过64个字节`。  


`64 - 19 = 45`，说好的`44`呢？  

原来是这样，`SDS`结构体中的`content`是以`NULL`结尾，而`NULL`占了一个字节。  


## 使用场景  

- 缓存  
- 计数器: 自增视频播放次数。  
- 限时限速： 5分钟内只能获取一次短信验证码。  


## `Reference`  
- `《Redis开发与运维》 - 2.2 字符串`  
- `《Redis深度历险 核心原理与应用实践》 - 5.1 丝分缕析--探索"字符串"内部`  