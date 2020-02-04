---
layout: post
title: Redis - 有序集合
category: redis
tags: [redis]
excerpt: Redis Sorted Set
---

## 常用命令  

#### 集合内  

- 增  

``` shell
# zadd key score member [score member ...]

zadd sorted_set_test 109 iPhone11 123 iPhone_XS_MAX 35 Honor_20S 215 iPhone_XR 21 Huawei_Mate_30
(integer) 4
127.0.0.1:6379> zscan sorted_set_test 0
1) "0"
2)  1) "Huawei_Mate_30"
    2) "21"
    3) "Honor_20S"
    4) "35"
    5) "iPhone11"
    6) "109"
    7) "iPhone_XS_MAX"
    8) "123"
    9) "iPhone_XR"
   10) "215"
```

- 删  

``` shell
# 1. zrem key element [element...]
127.0.0.1:6379> zrange sorted_set_test 0 -1
1) "yyc"
2) "11111111112222222222333333333344444444445555555555666666666677777"
3) "Huawei_Mate_30"
4) "Honor_20S"
5) "iPhone11"
6) "iPhone_XS_MAX"
7) "iPhone_XR"
127.0.0.1:6379> zrem sorted_set_test Huawei_Mate_30
(integer) 1
127.0.0.1:6379> zrange sorted_set_test 0 -1
1) "yyc"
2) "11111111112222222222333333333344444444445555555555666666666677777"
3) "Honor_20S"
4) "iPhone11"
5) "iPhone_XS_MAX"
6) "iPhone_XR"

# 2. zremrangebyrank key start end
127.0.0.1:6379> zrange sorted_set_test 0 -1 withscores
 1) "yyc"
 2) "1"
 3) "11111111112222222222333333333344444444445555555555666666666677777"
 4) "2"
 5) "Honor_20S"
 6) "35"
 7) "iPhone11"
 8) "109"
 9) "iPhone_XS_MAX"
10) "123"
11) "iPhone_XR"
12) "215"
127.0.0.1:6379> zremrangebyrank sorted_set_test 0 3
(integer) 4
127.0.0.1:6379> zrange sorted_set_test 0 -1 withscores
1) "iPhone_XS_MAX"
2) "123"
3) "iPhone_XR"
4) "215"

# 3. zremrangebyscore key min max
127.0.0.1:6379> zrange sorted_set_test 0 -1 withscores
1) "Honor_20S"
2) "35"
3) "iPhone11"
4) "109"
5) "iPhone_XS_MAX"
6) "123"
7) "iPhone_XR"
8) "215"
127.0.0.1:6379> zremrangebyscore sorted_set_test 0 100
(integer) 1
127.0.0.1:6379> zrange sorted_set_test 0 -1 withscores
1) "iPhone11"
2) "109"
3) "iPhone_XS_MAX"
4) "123"
5) "iPhone_XR"
6) "215"
```

- 改  

增加成员的分数：  

``` shell
127.0.0.1:6379> zscan sorted_set_test 0
1) "0"
2)  1) "Huawei_Mate_30"
    2) "21"
    3) "Honor_20S"
    4) "35"
    5) "iPhone11"
    6) "109"
    7) "iPhone_XS_MAX"
    8) "123"
    9) "iPhone_XR"
   10) "215"
# zincrby key increment member
127.0.0.1:6379> zincrby sorted_set_test 5 Huawei_Mate_30
"26"
```

- 查  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/query_methods.png)

``` shell
127.0.0.1:6379> zscan sorted_set_test 0
1) "0"
2)  1) "Huawei_Mate_30"
    2) "21"
    3) "Honor_20S"
    4) "35"
    5) "iPhone11"
    6) "109"
    7) "iPhone_XS_MAX"
    8) "123"
    9) "iPhone_XR"
   10) "215"

# 1. Calculate the number of members.
# zcard key
127.0.0.1:6379> zcard sorted_set_test
(integer) 5

# 2. Calculate the score of the member.
# zscore key member
127.0.0.1:6379> zscore sorted_set_test iPhone11
"109"

# 3. Calculate the rank of the member(begin with 0!).
# zrank key member
127.0.0.1:6379> zrank sorted_set_test iPhone11
(integer) 2

# 4. Return members in the specified ranking scope.
# zrange key start end [withscores]
# zrevrange key start end [withscores]
127.0.0.1:6379> zrange sorted_set_test 0 -1
1) "Huawei_Mate_30"
2) "Honor_20S"
3) "iPhone11"
4) "iPhone_XS_MAX"
5) "iPhone_XR"
127.0.0.1:6379> zrange sorted_set_test 0 -1 withscores
 1) "Huawei_Mate_30"
 2) "26"
 3) "Honor_20S"
 4) "35"
 5) "iPhone11"
 6) "109"
 7) "iPhone_XS_MAX"
 8) "123"
 9) "iPhone_XR"
10) "215"
127.0.0.1:6379> zrevrange sorted_set_test 0 -1 withscores
 1) "iPhone_XR"
 2) "215"
 3) "iPhone_XS_MAX"
 4) "123"
 5) "iPhone11"
 6) "109"
 7) "Honor_20S"
 8) "35"
 9) "Huawei_Mate_30"
10) "26"
127.0.0.1:6379> zrevrange sorted_set_test 0 2 withscores
1) "iPhone_XR"
2) "215"
3) "iPhone_XS_MAX"
4) "123"
5) "iPhone11"
6) "109"

# 5. Return members in the specified score scope.
# zrangebyscore key min max [withscores]
# zrevrangebyscore key max min [withscores]
127.0.0.1:6379> zrangebyscore sorted_set_test 100 200
1) "iPhone11"
2) "iPhone_XS_MAX"
127.0.0.1:6379> zrangebyscore sorted_set_test 100 200 withscores
1) "iPhone11"
2) "109"
3) "iPhone_XS_MAX"
4) "123"
127.0.0.1:6379> zrevrangebyscore sorted_set_test 200 100 withscores
1) "iPhone_XS_MAX"
2) "123"
3) "iPhone11"
4) "109"

# 6. Return the number of members in the specified score scope.
# zcount key min max
127.0.0.1:6379> zcount sorted_set_test 100 200
(integer) 2
```


#### 集合间  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/user_ranking_1_2.png)

> zinterstore destination numkeys key [key ...] [weights weight [weight ...]]
[agregate sum|min|max]  
zunionstore destination numkeys key [key ...] [weights weight [weight ...]]
[aggregate sum|min|max]  

以`zinterstore`为例：  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/zinter_max_min_sum.png)


``` shell
127.0.0.1:6379> zadd user:ranking:1 1 kris 91 mike 200 frank 220 tim 250 martin 251 tom
(integer) 6
127.0.0.1:6379> zadd user:ranking:2 8 james 77 mike 625 martin 888 tom
(integer) 4
127.0.0.1:6379> zinterstore sorted_set_inter_test 2 user:ranking:1 user:ranking:2 weights 1 0.5 aggregate max
(integer) 3
127.0.0.1:6379> zrange sorted_set_inter_test 0 -1 withscores
1) "mike"
2) "91"
3) "martin"
4) "312.5"
5) "tom"
6) "444"
127.0.0.1:6379> zinterstore sorted_set_inter_test_min 2 user:ranking:1 user:ranking:2 weights 1 0.5 aggregate min
(integer) 3
127.0.0.1:6379> zrange sorted_set_inter_test_min 0 -1 withscores
1) "mike"
2) "38.5"
3) "martin"
4) "250"
5) "tom"
6) "251"
127.0.0.1:6379> zinterstore sorted_set_inter_test_sum 2 user:ranking:1 user:ranking:2 weights 1 0.5 aggregate sum
(integer) 3
127.0.0.1:6379> zrange sorted_set_inter_test_sum 0 -1 withscores
1) "mike"
2) "129.5"
3) "martin"
4) "562.5"
5) "tom"
6) "695"
```

## 内部编码  

有序集合有两种内部编码类型：  
- `ziplist`  
- `skiplist`  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/sorted_set_encoding.png)  


证明：  

``` shell
127.0.0.1:6379> zadd sorted_set_test 1 yyc
(integer) 1
127.0.0.1:6379> object encoding sorted_set_test
"ziplist"
127.0.0.1:6379> zadd  sorted_set_test 2 11111111112222222222333333333344444444445555555555666666666677777
(integer) 1
127.0.0.1:6379> object encoding sorted_set_test
"skiplist"

127.0.0.1:6379> zcount sorted_set_test2 1 128
(integer) 128
127.0.0.1:6379> object encoding sorted_set_test2
"ziplist"
127.0.0.1:6379> zadd sorted_set_test2 129 129
(integer) 1
127.0.0.1:6379> object encoding sorted_set_test2
"skiplist"
```

## 使用场景  

- `排行榜`  


## `Redis`五种常用数据结构的内部编码  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/five_type_encoding.png)


## `Reference`  

- `《Redis开发与运维》 - 2.6 有序集合`