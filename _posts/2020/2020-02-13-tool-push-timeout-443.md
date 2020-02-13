---
layout: post
title: 工具 - Github Push Timeout 443
category: tool
tags: [tool]
excerpt: Github Push Timeout 443
---

本文主要记录下`Github Push timeout 443`问题：  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/push_timeout_443.png)  

解决方案：

`git config --global http.proxy "localhost:1080"`

来理解下这段语句的含义：  

- `--global`  

这代表使用了哪里的配置信息，主要分为三个地方：  

``` shell
--global    use global config file
--system    use system config file
--local     use repository config file
```

- `http.proxy "localhost:1080"`

这其实代表了一个键值对，`Key`是`http.proxy`，`Value`是`"localhost:1080"`。  

- 那么我们如何查询所有全局信息？  

``` shell
$ git config --global --list
user.email=15900617310@163.com
user.name=yycer
http.proxy=localhost:1080
```

- 那又如何删除其中的一个键值对？  

``` shell
$ git config --global user.age 25

$ git config --global --list
user.email=15900617310@163.com
user.name=yycer
user.age=25
http.proxy=localhost:1080

$ git config --global user.age
25

$ git config --global --unset user.age

$ git config --global --list
user.email=15900617310@163.com
user.name=yycer
http.proxy=localhost:1080
```

## `Reference`
- [Git push时报错：Failed connect to github.com:443](https://blog.csdn.net/qq_27093465/article/details/71210203){:target="_blank"}  
- [GIT-查看config配置信息](https://www.cnblogs.com/merray/p/6006411.html){:target="_blank"}  
- [使用github出了些问题?](https://www.zhihu.com/question/26954892){:target="_blank"}  

