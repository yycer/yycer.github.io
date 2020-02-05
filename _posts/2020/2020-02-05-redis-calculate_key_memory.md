---
layout: post
title: Redis - 计算键的内存
category: redis
tags: [redis]
excerpt: Redis Calculate Key Memory
---

## `Before Redis 4.0`  

以下测试环境为`Ubuntu 16.04 LTS`：  

- 安装`Python`  
- 安装`Pip`  
- 安装`rdbtools`: `pip install rdbtools python-lzf`  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/redis_3.2.6.png)  

<br>
![](https://yyc-images.oss-cn-beijing.aliyuncs.com/redis_memory_for_key.png)  



## `After Redis 4.0`  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/redis_5.0.7.png)  

直接使用`memory usage key`命令：  

``` shell
yyc16@ubuntu:~$ redis-cli -p 6379
127.0.0.1:6379> set name yyc
OK
127.0.0.1:6379> memory usage name
(integer) 51

```

## `Reference`
- [redis-rdb-tools](https://github.com/sripathikrishnan/redis-rdb-tools#installing-rdbtools){:target="_blank"}  
- [finding out the memory occupied by a single key in redis](https://pingredis.blogspot.com/2017/02/finding-out-memory-occupied-by-single.html){:target="_blank"}  
- [Redis: Show database size/size for keys](https://stackoverflow.com/questions/7638542/redis-show-database-size-size-for-keys){:target="_blank"}  