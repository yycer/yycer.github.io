---
layout: post
title: redis常用命令整理
category: redis
tags: [redis]
excerpt: redis常用命令大全
---
## key  

- 如何查看所有键？
```
keys *(pattern) // Find all keys matching the given pattern.
scan 0          // Incrementally iterate the keys space.
```
- 两者的区别是什么？  
    - `keys *`: 返回所有的键，查询过程中可能会造成几秒的延迟。 
    - `scan 0`: 返回两个元素，第一个元素代表后续元素的游标，若为0，则表示所有元素都已输出，第二个元素为keys(默认显示`10`个key)。  

![keys_scan](http://px8rn4o1y.bkt.clouddn.com/keys_scan.png)

- 如何判断某个键是否存在？
```
exists key
```

- 如何重命名某个键？
```
rename key newkey
```

- 如何删除某个键？
```
del key
```

- 如何对某个键设置过期时间？
```
expire  key 1000 (seconds)
pexpire key 10000(milliseconds)
```

- 如何查看某个键还剩多久过期？
```
// ttl: time to live
// After Redis 2.8:
// The command return -2 if the key does not exist.
// The command return -1 if the key exist but not associated expire.
ttl key(seconds)
pttl key(milliseconds)
```

- 如何持久化某个即将过期的键？
```
persist key
```

- 如何获得某个键的数据类型？
```
type key
```

- 返回或保存给定列表、集合、有序集合 key 中经过排序的元素。
```
sort key desc
```

<br>
## String  

- 如何为某个字符串赋值？
```
set  key "value"
mset key1 "v1" key2 "v2"
```

- 如何获得某个字符串的值？
```
get  key
mget key1 key2
```

- 如何获得某个字符串的长度？
```
strlen key
```

- 如何将值追加到key原来值的末尾？
```
append key "new_value"
```

- 如何自增/自减？
```
incr key
desc key
```

- 如何计算某一字符串对应二进制中1的个数？
```
bitcount key [start] [end]
```

- 如何对某一字符串做到既赋值，又设置过期时间？
```
setex  key [expiredTime] [new_value]
psetex key [expiredTime] [new_value]
```

- 如何对某一字符串既设置新值，又返回旧值？
```
getset key "new_value" // return "old_value"
```

- 将key的值设为value，当且仅当key不存在。
```
// setnx: Set if not exists.
setnx key "value"
```


<br>
## Hash  

- 如何创建Hash，并为其`field`(域)设置值？
```
hset  key field value
hmset key f1 v1 f2 v2
```

- 如何获得某个Hash中某个域的值？
```
hget  key field
hmget key f1 f2
```

- 如何获得某个Hash中所有键值对？
```
hgetall key
hscan key 0
```

- 如何获得某个Hash中键值对的个数？
```
hlen key
```

- 如何判断某个域是否存在于某个Hash中？
```
hexists key field1
```

- 如何删除某个Hash中的某个域？
```
hdel key field1
```

- 当某个Hash中某个field设置成value，当且仅当field不存在。
```
hsetnx key field value
```

<br>
## List  

- 如何往List中添加元素？
```
// 左上右下
lpush key value1
rpush key value2
```

- 如何得到List中第一个元素？
```
lpop key
```

- 如何得到List中最后一个元素？
```
rpop key
```

- 如何删除List中的元素？
```
// 1. count > 0, 从表头往表尾搜索, 移除值为value的元素, 数量为count。
// 2. count < 0, 从表尾往表头搜索, 移除值为value的元素, 数量为count的绝对值。
// 3. count = 0, 移除值为value的所有元素。
lrem key count value
```

- 如何获得List中所有元素？
```
lrange key [start] [end]
```

- 获得获得List中元素的数量？
```
llen key
```

- 如何获得List中指定索引的元素？
```
lindex key index
```

- 原子操作
```
// 将source中表尾元素，插入destination的表头位置。
rpoplpush source destination
```


<br>
## Set  

- 如何往Set中添加一个元素？
```
sadd key v1 v2 v3
```

- 如何删除Set中某个元素？ 
```
srem key member [member...]
```

- 如何获得Set中所有元素？
```
sscan key 0
smembers key
```

- 如何判断Set是否包含某个元素？
```
sismember key value
```

- 如何获得Set的元素数量？
```
scard key
```

- 如何获得两个Set的交集？
```
sinter key1 key2
```

- 如何获得两个Set的并集？
```
sunion key1 key2
```

- 如何获得两个Set的差集？
```
sdiff key1 key2 // key1中独有
sdiff key2 key1 // key2中独有
```


<br>
## SortedSet  

- 如何往SortedSet中添加成员？
```
zadd key score member [[score member] [score member]...]
```

- 如何删除SortedSet中某个成员。
```
zrem key member [member...]
zremrangebyscore key min max   // 删除score范围内的成员，redis 2.1.6后删除的元素不包括min和max。
zremrangebyrank  key start stop // 根据score正序排序，删除前stop - start + 1个成员。
```

- 如何获得SortedSet中所有成员？
```
zscan     key 0
zrange    key 0 -1 withscores
zrevrange key 0 -1 withscores
```

- 如何根据score排列？
```
zrangebyscore    page min max
zrevrangebyscore page max min // 注意是先max，再min喔。
```

- 如何获得某一成员的排名？
```
zrank    key member
zrevrank key member
```

- 如何获得SortedSet中的元素数量？
```
zcard  key
zcount key min max
```

- 如何获得某个成员的分数？
```
zscore key member
```
