---
layout: post
title: Redis - 集群搭建
category: redis
tags: [redis]
excerpt: Redis Cluster Building
---

这篇文章记录下如何搭建集群，主要分为两种方式，如下所示：  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/build_cluster.png)  

## 手工搭建集群  

`Redis`集群一般至少由`6`个节点组成，目的就是为了保证高可用。  


####  准备并启动所有节点  

首先准备`conf`配置，文件命名规则： `redis-{port}.conf`  

``` shell
port 6379
cluster-enabled yes
cluster-node-timeout 15000
cluster-config-file "nodes-6379.conf"
```

启动所有节点：  

``` shell
redis-server redis-6379.conf & 
redis-server redis-6380.conf & 
redis-server redis-6381.conf & 
redis-server redis-6382.conf & 
redis-server redis-6383.conf & 
redis-server redis-6384.conf
```

####  节点握手  

节点握手是指： 一批在集群下的节点通过`Gossip`协议彼此通信，感知对方。  


``` shell
127.0.0.1:6379> cluster nodes
c5e0f824277907e304660b649dcf04933668d532 :6379 myself,master - 0 0 0 connected

127.0.0.1:6379> cluster meet 6380
(error) ERR Wrong CLUSTER subcommand or number of arguments
127.0.0.1:6379> cluster meet 127.0.0.1 6380
OK
127.0.0.1:6379> cluster meet 127.0.0.1 6381
OK
127.0.0.1:6379> cluster meet 127.0.0.1 6382
OK
127.0.0.1:6379> cluster meet 127.0.0.1 6383
OK
127.0.0.1:6379> cluster meet 127.0.0.1 6384
OK
127.0.0.1:6379> cluster nodes
c5e0f824277907e304660b649dcf04933668d532 127.0.0.1:6379 myself,master - 0 0 1 connected
686100a32ccbb316c040535341b870a4d7abe7b5 127.0.0.1:6383 master - 0 1580526377262 0 connected
cb8a1c88ead5283b3a9878450d36d86ca2918a00 127.0.0.1:6381 master - 0 1580526373233 2 connected
aae3785a6e2248d8008a2577d0c1df9786d6772e 127.0.0.1:6382 master - 0 1580526376254 3 connected
f6304c4667b960c68ae599c757b25db5715ab7ec 127.0.0.1:6380 master - 0 1580526378269 0 connected
c805eaa41747e209a05ec042eb1b78c8ae369b9e 127.0.0.1:6384 master - 0 1580526379278 4 connected

```

####  主节点分配槽  

``` shell
yyc16@ubuntu:~$ redis-cli -h 127.0.0.1 -p 6379 cluster addslots {0..5461}
OK
yyc16@ubuntu:~$ redis-cli -h 127.0.0.1 -p 6380 cluster addslots {5462..10922}
OK
yyc16@ubuntu:~$ redis-cli -h 127.0.0.1 -p 6381 cluster addslots {10923..16383}
OK

127.0.0.1:6379> cluster nodes
686100a32ccbb316c040535341b870a4d7abe7b5 127.0.0.1:6383 master - 0 1580527236692 5 connected
aae3785a6e2248d8008a2577d0c1df9786d6772e 127.0.0.1:6382 master - 0 1580527234674 3 connected
f6304c4667b960c68ae599c757b25db5715ab7ec 127.0.0.1:6380 master - 0 1580527235683 0 connected 5462-10922
c805eaa41747e209a05ec042eb1b78c8ae369b9e 127.0.0.1:6384 master - 0 1580527238706 4 connected
c5e0f824277907e304660b649dcf0933668d532 127.0.0.1:6379 myself,master - 0 0 1 connected 0-5461
cb8a1c88ead5283b3a9878450d36d86ca2918a00 127.0.0.1:6381 master - 0 1580527237699 2 connected 10923-16383
```

####  设置从节点  

``` shell
yyc16@ubuntu:~$ redis-cli -p 6382
127.0.0.1:6382> cluster replicate c5e0f824277907e304660b649dcf04933668d532
OK
yyc16@ubuntu:~$ redis-cli -p 6383
127.0.0.1:6383> cluster replicate f6304c4667b960c68ae599c757b25db5715ab7ec
OK
yyc16@ubuntu:~$ redis-cli -p 6384
127.0.0.1:6384> cluster replicate cb8a1c88ead5283b3a9878450d36d86ca2918a00
OK

yyc16@ubuntu:~$ redis-cli -p 6379
127.0.0.1:6379> cluster nodes
686100a32ccbb316c040535341b870a4d7abe7b5 127.0.0.1:6383 slave f6304c4667b960c68ae599c757b25db5715ab7ec 0 1580527448118 5 connected
aae3785a6e2248d8008a2577d0c1df9786d6772e 127.0.0.1:6382 slave c5e0f824277907e304660b649dcf04933668d532 0 1580527445099 3 connected
f6304c4667b960c68ae599c757b25db5715ab7ec 127.0.0.1:6380 master - 0 1580527449125 0 connected 5462-10922
c805eaa41747e209a05ec042eb1b78c8ae369b9e 127.0.0.1:6384 slave cb8a1c88ead5283b3a9878450d36d86ca2918a00 0 1580527447114 4 connected
c5e0f824277907e304660b649dcf04933668d532 127.0.0.1:6379 myself,master - 0 0 1 connected 0-5461
cb8a1c88ead5283b3a9878450d36d86ca2918a00 127.0.0.1:6381 master - 0 1580527446106 2 connected 10923-16383
```

整个集群的结构如下图所示：  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/cluster.png)  


## 工具搭建集群  

`redis-trib.rb`是用`Ruby`实现的`Redis`集群管理工具，作用就是简化集群搭建过程。  

提前准备：  

``` shell
sudo apt-get install ruby
sudo gem install redis
cp /home/yyc16/redis-3.2.6/src/redis-trib.rb /usr/local/bin/
```

第一步还是准备并启动所有节点：  

``` shell
redis-server redis-6379.conf & 
redis-server redis-6380.conf & 
redis-server redis-6381.conf & 
redis-server redis-6382.conf & 
redis-server redis-6383.conf & 
redis-server redis-6384.conf
```

第二步，就可以使用`redis-trib.rb create`命令完成节点握手、槽分配和设置从节点。  


``` shell

yyc16@ubuntu:~$ redis-trib.rb create --replicas 1 127.0.0.1:6379 127.0.0.1:6380 127.0.0.1:6381 127.0.0.1:6382 127.0.0.1:6383 127.0.0.1:6384
>>> Creating cluster
>>> Performing hash slots allocation on 6 nodes...
Using 3 masters:
127.0.0.1:6379
127.0.0.1:6380
127.0.0.1:6381
Adding replica 127.0.0.1:6382 to 127.0.0.1:6379
Adding replica 127.0.0.1:6383 to 127.0.0.1:6380
Adding replica 127.0.0.1:6384 to 127.0.0.1:6381
M: ebb10da2ec13b1da08037a81fbd41204023b898c 127.0.0.1:6379
   slots:0-5460 (5461 slots) master
M: dbccf4e50f84cb35a605945e284af93cdc216ff8 127.0.0.1:6380
   slots:5461-10922 (5462 slots) master
M: 189cec3cf8a1f10fea39c56f92db348936bca4ab 127.0.0.1:6381
   slots:10923-16383 (5461 slots) master
S: 72f173b5604676a6f02d3be5f364210b39eeb09f 127.0.0.1:6382
   replicates ebb10da2ec13b1da08037a81fbd41204023b898c
S: 1b66493753d96aadf7b77d75b7903a0878b922df 127.0.0.1:6383
   replicates dbccf4e50f84cb35a605945e284af93cdc216ff8
S: 0cb756c4efde229af614c93b847ad5477de75bc5 127.0.0.1:6384
   replicates 189cec3cf8a1f10fea39c56f92db348936bca4ab
Can I set the above configuration? (type 'yes' to accept): yes

>>> Nodes configuration updated
>>> Assign a different config epoch to each node
>>> Sending CLUSTER MEET messages to join the cluster
Waiting for the cluster to join.....
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


127.0.0.1:6379> cluster nodes
72f173b5604676a6f02d3be5f364210b39eeb09f 127.0.0.1:6382 slave ebb10da2ec13b1da08037a81fbd41204023b898c 0 1580541110945 4 connected
dbccf4e50f84cb35a605945e284af93cdc216ff8 127.0.0.1:6380 master - 0 1580541109937 2 connected 5461-10922
189cec3cf8a1f10fea39c56f92db348936bca4ab 127.0.0.1:6381 master - 0 1580541112961 3 connected 10923-16383
0cb756c4efde229af614c93b847ad5477de75bc5 127.0.0.1:6384 slave 189cec3cf8a1f10fea39c56f92db348936bca4ab 0 1580541111954 6 connected
1b66493753d96aadf7b77d75b7903a0878b922df 127.0.0.1:6383 slave dbccf4e50f84cb35a605945e284af93cdc216ff8 0 1580541113969 5 connected
ebb10da2ec13b1da08037a81fbd41204023b898c 127.0.0.1:6379 myself,master - 0 0 1 connected 0-5460
```

最后再来说下，如何检查集群的完整性？  

所谓集群完整性就是：所有`16384[2^14]`个槽都要分配给存活的主节点。  


``` shell
yyc16@ubuntu:~$ redis-trib.rb check 127.0.0.1:6379
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
```

## `Reference`
- `《Redis开发与运维》 - 10.2 搭建集群`
- [ubuntu安装redis集群](https://www.jianshu.com/p/0dc64585a2a9){:target="_blank"}