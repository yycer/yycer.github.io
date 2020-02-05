---
layout: post
title: Redis - 理解内存
category: redis
tags: [redis]
excerpt: Redis Memory Understanding
---

本文主要以下三个方面理解`Redis`内存：  

- `内存消耗`  
- `内存回收`  
- `内存优化`  


## 内存消耗  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/redis_memory_consume.png)  


## 内存回收  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/redis_memory_recycle.png)  


## 内存优化  

先来介绍下`redisObject`对象：  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/redis_memory_redisobject.png)  

然后再介绍下`"共享整数池"`：  

为什么`Redis`要维护一个`[0-9999]`的共享整数池？  

节省内存，因为当你创建大量整数时，`redisObject`也是一笔很大的开销，甚至超过了整数本身。  

好，我们来实践一下：  

``` shell
127.0.0.1:6379> set foo 100
OK
127.0.0.1:6379> object refcount foo
(integer) 2
127.0.0.1:6379> set bar 100
OK
127.0.0.1:6379> object refcount bar
(integer) 3
```

但有个特殊情况：当你设置了`maxmemory`，并且`内存回收策略`设置成与`LRU`算法相关时，`Redis`禁止使用共享整数池。  


``` shell
127.0.0.1:6379> set key:1 120
OK
127.0.0.1:6379> object refcount key:1
(integer) 2
127.0.0.1:6379> config get maxmemory-policy
1) "maxmemory-policy"
2) "noeviction"
127.0.0.1:6379> config set maxmemory-policy volatile-lru
OK
127.0.0.1:6379> set key:2 120
OK
127.0.0.1:6379> object refcount key:2
(integer) 3
127.0.0.1:6379> config set maxmemory 1GB
OK
127.0.0.1:6379> config get maxmemory
1) "maxmemory"
2) "1073741824"
127.0.0.1:6379> set key:3 120
OK
127.0.0.1:6379> object refcount key:3
(integer) 1
127.0.0.1:6379> config set maxmemory-policy noeviction
OK
127.0.0.1:6379> set key:4 120
OK
127.0.0.1:6379> object refcount key:4
(integer) 4

```

最后来介绍下其他`四种`内存优化方法：  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/redis_memory_optimization.png)  



## `Reference`  

- `《Redis开发与运维》 - 8 理解内存`  

