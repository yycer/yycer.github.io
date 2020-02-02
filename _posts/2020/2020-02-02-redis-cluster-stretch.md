---
layout: post
title: Redis - 集群伸缩
category: redis
tags: [redis]
excerpt: Redis Cluster Stretch
---

## 集群扩容  

先来张蓝图：  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/redis_stretch_2.png)   


#### 准备新节点  

首先提前准备好新节点配置信息：  

``` shell
port 6385
cluster-enabled yes
cluster-node-timeout 15000
cluster-config-file "nodes-6385.conf"
```

然后启动所有新增节点：  

``` shell
redis-server redis-6385.conf & 
redis-server redis-6386.conf & 
```

#### 加入集群  

一共有两种方式：

- `cluster meet host:port`  
- `redis-trib.rb add-node ...`  


推荐使用第二种，原因是因为：`防止加入节点的集群被合并到现有集群`。  

再来看下`redis-trib.rb add-node`针对主从节点的不同语法：  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/redis_trib_add_node.png)  


``` shell
# Step1: Add master node 6385
yyc16@ubuntu:~/redis_cluster_test/conf$ redis-trib.rb add-node 127.0.0.1:6385 127.0.0.1:6379
>>> Adding node 127.0.0.1:6385 to cluster 127.0.0.1:6379
>>> Performing Cluster Check (using node 127.0.0.1:6379)
M: ebb10da2ec13b1da08037a81fbd41204023b898c 127.0.0.1:6379
   slots:0-5460 (5461 slots) master
   1 additional replica(s)
S: 72f173b5604676a6f02d3be5f364210b39eeb09f 127.0.0.1:6382
   slots: (0 slots) slave
   replicates ebb10da2ec13b1da08037a81fbd41204023b898c
M: dbccf4e50f84cb35a605945e284af93cdc216ff8 127.0.0.1:6380
   slots:5461-10922 (5462 slots) master
   1 additional replica(s)
M: 189cec3cf8a1f10fea39c56f92db348936bca4ab 127.0.0.1:6381
   slots:10923-16383 (5461 slots) master
   1 additional replica(s)
S: 0cb756c4efde229af614c93b847ad5477de75bc5 127.0.0.1:6384
   slots: (0 slots) slave
   replicates 189cec3cf8a1f10fea39c56f92db348936bca4ab
S: 1b66493753d96aadf7b77d75b7903a0878b922df 127.0.0.1:6383
   slots: (0 slots) slave
   replicates dbccf4e50f84cb35a605945e284af93cdc216ff8
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.
>>> Send CLUSTER MEET to node 127.0.0.1:6385 to make it join the cluster.
[OK] New node added correctly.

# Step2: Add slave node 6386
redis-trib.rb add-node --slave --master-id 10f5eb94d2aee95100a64e04b307e8c54c74d7e0 127.0.0.1:6386 127.0.0.1:6379
...

127.0.0.1:6379> cluster nodes
72f173b5604676a6f02d3be5f364210b39eeb09f 127.0.0.1:6382 slave ebb10da2ec13b1da08037a81fbd41204023b898c 0 1580558691275 4 connected
63f02b0c33712e08232ee766a5e87858ea5f2313 127.0.0.1:6386 slave 10f5eb94d2aee95100a64e04b307e8c54c74d7e0 0 1580558695295 0 connected
dbccf4e50f84cb35a605945e284af93cdc216ff8 127.0.0.1:6380 master - 0 1580558696302 2 connected 5461-10922
10f5eb94d2aee95100a64e04b307e8c54c74d7e0 127.0.0.1:6385 master - 0 1580558692280 0 connected
189cec3cf8a1f10fea39c56f92db348936bca4ab 127.0.0.1:6381 master - 0 1580558697306 3 connected 10923-16383
0cb756c4efde229af614c93b847ad5477de75bc5 127.0.0.1:6384 slave 189cec3cf8a1f10fea39c56f92db348936bca4ab 0 1580558694291 6 connected
1b66493753d96aadf7b77d75b7903a0878b922df 127.0.0.1:6383 slave dbccf4e50f84cb35a605945e284af93cdc216ff8 0 1580558695295 5 connected
ebb10da2ec13b1da08037a81fbd41204023b898c 127.0.0.1:6379 myself,master - 0 0 1 connected 0-5460
```

#### 迁移槽和数据  

先来整理下手工迁移的过程：  


以下图片引用自`《Redis开发与运维》 P296 - 槽和数据迁移流程`

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/redis_stretch_migrate_slot_and_data.png)  


``` shell
# Step1: 目标节点[6385]准备导入槽。
127.0.0.1:6385> cluster setslot 4096 importing ebb10da2ec13b1da08037a81fbd41204023b898c
OK
127.0.0.1:6385> cluster nodes
dbccf4e50f84cb35a605945e284af93cdc216ff8 127.0.0.1:6380 master - 0 1580558880667 2 connected 5461-10922
ebb10da2ec13b1da08037a81fbd41204023b898c 127.0.0.1:6379 master - 0 1580558882174 1 connected 0-5460
0cb756c4efde229af614c93b847ad5477de75bc5 127.0.0.1:6384 slave 189cec3cf8a1f10fea39c56f92db348936bca4ab 0 1580558884686 3 connected
1b66493753d96aadf7b77d75b7903a0878b922df 127.0.0.1:6383 slave dbccf4e50f84cb35a605945e284af93cdc216ff8 0 1580558881169 2 connected
72f173b5604676a6f02d3be5f364210b39eeb09f 127.0.0.1:6382 slave ebb10da2ec13b1da08037a81fbd41204023b898c 0 1580558883179 1 connected
63f02b0c33712e08232ee766a5e87858ea5f2313 127.0.0.1:6386 slave 10f5eb94d2aee95100a64e04b307e8c54c74d7e0 0 1580558884183 0 connected
189cec3cf8a1f10fea39c56f92db348936bca4ab 127.0.0.1:6381 master - 0 1580558880165 3 connected 10923-16383
10f5eb94d2aee95100a64e04b307e8c54c74d7e0 127.0.0.1:6385 myself,master - 0 0 0 connected [4096-<-ebb10da2ec13b1da08037a81fbd41204023b898c]

# Step2: 源节点[6379]准备导出槽。
127.0.0.1:6379> cluster setslot 4096 migrating 10f5eb94d2aee95100a64e04b307e8c54c74d7e0
OK
127.0.0.1:6379> cluster nodes
72f173b5604676a6f02d3be5f364210b39eeb09f 127.0.0.1:6382 slave ebb10da2ec13b1da08037a81fbd41204023b898c 0 1580558936417 4 connected
63f02b0c33712e08232ee766a5e87858ea5f2313 127.0.0.1:6386 slave 10f5eb94d2aee95100a64e04b307e8c54c74d7e0 0 1580558935412 0 connected
dbccf4e50f84cb35a605945e284af93cdc216ff8 127.0.0.1:6380 master - 0 1580558933404 2 connected 5461-10922
10f5eb94d2aee95100a64e04b307e8c54c74d7e0 127.0.0.1:6385 master - 0 1580558930894 0 connected
189cec3cf8a1f10fea39c56f92db348936bca4ab 127.0.0.1:6381 master - 0 1580558934910 3 connected 10923-16383
0cb756c4efde229af614c93b847ad5477de75bc5 127.0.0.1:6384 slave 189cec3cf8a1f10fea39c56f92db348936bca4ab 0 1580558934409 6 connected
1b66493753d96aadf7b77d75b7903a0878b922df 127.0.0.1:6383 slave dbccf4e50f84cb35a605945e284af93cdc216ff8 0 1580558937421 5 connected
ebb10da2ec13b1da08037a81fbd41204023b898c 127.0.0.1:6379 myself,master - 0 0 1 connected 0-5460 [4096->-10f5eb94d2aee95100a64e04b307e8c54c74d7e0]

# Step3: 从源节点[6379]slot下获取count个键。
127.0.0.1:6379> cluster getkeysinslot 4096 100
1) "k25019"
2) "k34058"
3) "k55485"
127.0.0.1:6379> mget k25019 k34058 k55485
1) "v25019"
2) "v34058"
3) "v55485"

# Step4 & 5: 批量迁移键...
127.0.0.1:6379> migrate 127.0.0.1 6385 "" 0 5000 keys k25019 k34058 k55485
OK
127.0.0.1:6379> mget k25019 k34058 k55485
(error) ASK 4096 127.0.0.1:6385

# Step6: 所有主节点被通知槽已经分配给目标节点[6385]了。
127.0.0.1:6379> cluster setslot 4096 node 10f5eb94d2aee95100a64e04b307e8c54c74d7e0
OK
yyc16@ubuntu:~$ redis-cli -p 6380
127.0.0.1:6380> 
127.0.0.1:6380> cluster setslot 4096 node 10f5eb94d2aee95100a64e04b307e8c54c74d7e0
OK
127.0.0.1:6380> 
yyc16@ubuntu:~$ redis-cli -p 6381
127.0.0.1:6381> cluster setslot 4096 node 10f5eb94d2aee95100a64e04b307e8c54c74d7e0
OK
127.0.0.1:6381> 
yyc16@ubuntu:~$ redis-cli -p 6385
127.0.0.1:6385> cluster setslot 4096 node 10f5eb94d2aee95100a64e04b307e8c54c74d7e0
OK

127.0.0.1:6385> mget k25019 k34058 k55485
1) "v25019"
2) "v34058"
3) "v55485"
```

讲个有意思的地方：  

如何模拟往指定槽`eg: 4096`中添加键？  

想来想去，没想到好办法，就让程序替我算`10W`次吧~  

``` java
@Test
public void keySlot4096Test(){
    Jedis jedis = new Jedis("localhost", 6379);
    for (int i = 0; i <= 100000; i++){
        StringBuilder sb = new StringBuilder();
        sb.append("k");
        sb.append(i);
        String str = new String(sb);
        Long keySlot = jedis.clusterKeySlot(str);
        if (keySlot == 4096){
            System.out.println(str);
        }
    }
}
-------------  Console  --------------
// k25019
// k34058
// k55485
// k63334
// k66446
// k72375
// k77407
```


再来说下通过`redis-trib.rb reshard`命令来迁移槽和数据：  

- 先来看下集群的原始状态  

``` shell
127.0.0.1:6379> cluster nodes
d3acb0e0fbd9b1a43945bc528b7b04366648f3b3 127.0.0.1:6381 master - 0 1580560815476 3 connected 10923-16383
15736379a637bf70ebc4e384a59a016d4b1ffd9c 127.0.0.1:6380 master - 0 1580560814471 2 connected 5461-10922
e8cfef2e7b780246da44944d687478a97a31b919 127.0.0.1:6382 slave fea83e8194db220390d692f9dc7e8600ad89e39f 0 1580560811455 4 connected
32825fe8d1c57aa9ea2bbe3ec8f04c6d213a612b 127.0.0.1:6383 slave 15736379a637bf70ebc4e384a59a016d4b1ffd9c 0 1580560812459 5 connected
3dcaa8e5e31674d11fef8eb5f607029855125111 127.0.0.1:6384 slave d3acb0e0fbd9b1a43945bc528b7b04366648f3b3 0 1580560813465 6 connected
fea83e8194db220390d692f9dc7e8600ad89e39f 127.0.0.1:6379 myself,master - 0 0 1 connected 0-5460

```

- 准备并启动新节点  

``` shell
redis-server redis-6385.conf &
redis-server redis-6386.conf &
```

- 添加新节点  

``` shell
redis-trib.rb add-node 127.0.0.1:6385 127.0.0.1:6379

127.0.0.1:6379> cluster nodes
5c6d4eb0b343885b9d4f4a00558273df2de01e77 127.0.0.1:6385 master - 0 1580560951278 0 connected
d3acb0e0fbd9b1a43945bc528b7b04366648f3b3 127.0.0.1:6381 master - 0 1580560953188 3 connected 10923-16383
15736379a637bf70ebc4e384a59a016d4b1ffd9c 127.0.0.1:6380 master - 0 1580560951176 2 connected 5461-10922
e8cfef2e7b780246da44944d687478a97a31b919 127.0.0.1:6382 slave fea83e8194db220390d692f9dc7e8600ad89e39f 0 1580560952184 4 connected
32825fe8d1c57aa9ea2bbe3ec8f04c6d213a612b 127.0.0.1:6383 slave 15736379a637bf70ebc4e384a59a016d4b1ffd9c 0 1580560954192 5 connected
3dcaa8e5e31674d11fef8eb5f607029855125111 127.0.0.1:6384 slave d3acb0e0fbd9b1a43945bc528b7b04366648f3b3 0 1580560950165 6 connected
fea83e8194db220390d692f9dc7e8600ad89e39f 127.0.0.1:6379 myself,master - 0 0 1 connected 0-5460

redis-trib.rb add-node --slave --master-id 5c6d4eb0b343885b9d4f4a00558273df2de01e77 127.0.0.1:6386 127.0.0.1:6379

127.0.0.1:6379> cluster nodes
5c6d4eb0b343885b9d4f4a00558273df2de01e77 127.0.0.1:6385 master - 0 1580561097861 0 connected
d3acb0e0fbd9b1a43945bc528b7b04366648f3b3 127.0.0.1:6381 master - 0 1580561096858 3 connected 10923-16383
15736379a637bf70ebc4e384a59a016d4b1ffd9c 127.0.0.1:6380 master - 0 1580561095856 2 connected 5461-10922
e8cfef2e7b780246da44944d687478a97a31b919 127.0.0.1:6382 slave fea83e8194db220390d692f9dc7e8600ad89e39f 0 1580561093846 4 connected
32825fe8d1c57aa9ea2bbe3ec8f04c6d213a612b 127.0.0.1:6383 slave 15736379a637bf70ebc4e384a59a016d4b1ffd9c 0 1580561096356 5 connected
3dcaa8e5e31674d11fef8eb5f607029855125111 127.0.0.1:6384 slave d3acb0e0fbd9b1a43945bc528b7b04366648f3b3 0 1580561095353 6 connected
28580f48d9546c7dd2e027b1525d6ea8dcfb4204 127.0.0.1:6386 slave 5c6d4eb0b343885b9d4f4a00558273df2de01e77 0 1580561091836 0 connected
fea83e8194db220390d692f9dc7e8600ad89e39f 127.0.0.1:6379 myself,master - 0 0 1 connected 0-5460
```

- 重新分配槽和数据  

``` shell
redis-trib.rb reshard 127.0.0.1:6379

127.0.0.1:6379> cluster nodes
5c6d4eb0b343885b9d4f4a00558273df2de01e77 127.0.0.1:6385 master - 0 1580561190321 8 connected 0-1364 5461-6826 10923-12287
d3acb0e0fbd9b1a43945bc528b7b04366648f3b3 127.0.0.1:6381 master - 0 1580561185287 3 connected 12288-16383
15736379a637bf70ebc4e384a59a016d4b1ffd9c 127.0.0.1:6380 master - 0 1580561187306 2 connected 6827-10922
e8cfef2e7b780246da44944d687478a97a31b919 127.0.0.1:6382 slave fea83e8194db220390d692f9dc7e8600ad89e39f 0 1580561186301 4 connected
32825fe8d1c57aa9ea2bbe3ec8f04c6d213a612b 127.0.0.1:6383 slave 15736379a637bf70ebc4e384a59a016d4b1ffd9c 0 1580561189316 5 connected
3dcaa8e5e31674d11fef8eb5f607029855125111 127.0.0.1:6384 slave d3acb0e0fbd9b1a43945bc528b7b04366648f3b3 0 1580561188310 6 connected
28580f48d9546c7dd2e027b1525d6ea8dcfb4204 127.0.0.1:6386 slave 5c6d4eb0b343885b9d4f4a00558273df2de01e77 0 1580561184270 8 connected
fea83e8194db220390d692f9dc7e8600ad89e39f 127.0.0.1:6379 myself,master - 0 0 1 connected 1365-5460
```

- 检查集群完整性  

``` shell
yyc16@ubuntu:~$ redis-trib.rb check 127.0.0.1:6379
>>> Performing Cluster Check (using node 127.0.0.1:6379)
M: fea83e8194db220390d692f9dc7e8600ad89e39f 127.0.0.1:6379
   slots:1365-5460 (4096 slots) master
   1 additional replica(s)
M: 5c6d4eb0b343885b9d4f4a00558273df2de01e77 127.0.0.1:6385
   slots:0-1364,5461-6826,10923-12287 (4096 slots) master
   1 additional replica(s)
M: d3acb0e0fbd9b1a43945bc528b7b04366648f3b3 127.0.0.1:6381
   slots:12288-16383 (4096 slots) master
   1 additional replica(s)
M: 15736379a637bf70ebc4e384a59a016d4b1ffd9c 127.0.0.1:6380
   slots:6827-10922 (4096 slots) master
   1 additional replica(s)
S: e8cfef2e7b780246da44944d687478a97a31b919 127.0.0.1:6382
   slots: (0 slots) slave
   replicates fea83e8194db220390d692f9dc7e8600ad89e39f
S: 32825fe8d1c57aa9ea2bbe3ec8f04c6d213a612b 127.0.0.1:6383
   slots: (0 slots) slave
   replicates 15736379a637bf70ebc4e384a59a016d4b1ffd9c
S: 3dcaa8e5e31674d11fef8eb5f607029855125111 127.0.0.1:6384
   slots: (0 slots) slave
   replicates d3acb0e0fbd9b1a43945bc528b7b04366648f3b3
S: 28580f48d9546c7dd2e027b1525d6ea8dcfb4204 127.0.0.1:6386
   slots: (0 slots) slave
   replicates 5c6d4eb0b343885b9d4f4a00558273df2de01e77
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.
```

## 集群收缩  

集群收缩流程如下：  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/clsuer_whrink.png)  

我们模拟下线`6381`和`6384`节点，其中`6381`为主节点，需要迁移槽和数据，因此分别向`6379、6380、6385`节点迁移`1365、1365、1366`个槽。  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/cluster_shrink_2.png)   

- 迁移槽和数据  

``` shell
redis-trib.rb reshard 127.0.0.1:6381

How many slots do you want to move (from 1 to 16384)? 1365
...
What is the receiving node ID? fea83e8194db220390d692f9dc7e8600ad89e39f[6379]
...
Source node #1:d3acb0e0fbd9b1a43945bc528b7b04366648f3b3[6381]
Source node #2:done
...
Do you want to proceed with the proposed reshard plan (yes/no)? yes
...

127.0.0.1:6379> cluster nodes
5c6d4eb0b343885b9d4f4a00558273df2de01e77 127.0.0.1:6385 master - 0 1580644329901 11 connected 0-1364 5461-6826 10923-12287 15018-16383
d3acb0e0fbd9b1a43945bc528b7b04366648f3b3 127.0.0.1:6381 master - 0 1580644328898 3 connected
15736379a637bf70ebc4e384a59a016d4b1ffd9c 127.0.0.1:6380 master - 0 1580644327894 10 connected 6827-10922 13653-15017
e8cfef2e7b780246da44944d687478a97a31b919 127.0.0.1:6382 slave fea83e8194db220390d692f9dc7e8600ad89e39f 0 1580644326892 9 connected
32825fe8d1c57aa9ea2bbe3ec8f04c6d213a612b 127.0.0.1:6383 slave 15736379a637bf70ebc4e384a59a016d4b1ffd9c 0 1580644326389 10 connected
3dcaa8e5e31674d11fef8eb5f607029855125111 127.0.0.1:6384 slave 5c6d4eb0b343885b9d4f4a00558273df2de01e77 0 1580644330905 11 connected
28580f48d9546c7dd2e027b1525d6ea8dcfb4204 127.0.0.1:6386 slave 5c6d4eb0b343885b9d4f4a00558273df2de01e77 0 1580644328397 11 connected
fea83e8194db220390d692f9dc7e8600ad89e39f 127.0.0.1:6379 myself,master - 0 0 9 connected 1365-5460 12288-13652
```

- 删除节点  

``` shell
yyc16@ubuntu:~$ redis-trib.rb del-node 127.0.0.1:6379 d3acb0e0fbd9b1a43945bc528b7b04366648f3b3
>>> Removing node d3acb0e0fbd9b1a43945bc528b7b04366648f3b3 from cluster 127.0.0.1:6379
>>> Sending CLUSTER FORGET messages to the cluster...
>>> SHUTDOWN the node.
yyc16@ubuntu:~$ redis-trib.rb del-node 127.0.0.1:6379 3dcaa8e5e31674d11fef8eb5f607029855125111
>>> Removing node 3dcaa8e5e31674d11fef8eb5f607029855125111 from cluster 127.0.0.1:6379
>>> Sending CLUSTER FORGET messages to the cluster...
>>> SHUTDOWN the node.

yyc16@ubuntu:~$ redis-cli -p 6379
127.0.0.1:6379> cluster nodes
5c6d4eb0b343885b9d4f4a00558273df2de01e77 127.0.0.1:6385 master - 0 1580644916862 11 connected 0-1364 5461-6826 10923-12287 15018-16383
15736379a637bf70ebc4e384a59a016d4b1ffd9c 127.0.0.1:6380 master - 0 1580644914850 10 connected 6827-10922 13653-15017
e8cfef2e7b780246da44944d687478a97a31b919 127.0.0.1:6382 slave fea83e8194db220390d692f9dc7e8600ad89e39f 0 1580644917871 9 connected
32825fe8d1c57aa9ea2bbe3ec8f04c6d213a612b 127.0.0.1:6383 slave 15736379a637bf70ebc4e384a59a016d4b1ffd9c 0 1580644911839 10 connected
28580f48d9546c7dd2e027b1525d6ea8dcfb4204 127.0.0.1:6386 slave 5c6d4eb0b343885b9d4f4a00558273df2de01e77 0 1580644915856 11 connected
fea83e8194db220390d692f9dc7e8600ad89e39f 127.0.0.1:6379 myself,master - 0 0 9 connected 1365-5460 12288-13652
```

## `Reference`
- `《Redis开发与运维》 - 10.4 集群伸缩`