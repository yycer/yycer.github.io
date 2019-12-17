---
layout: post
title: 按位异或
category: computer
tags: [computer]
excerpt: XOR
---

这篇文章主要讲下如何通过取反`!`、按位与`&`、按位或`|`，推导出按位异或`^`。  

先来看张表格：  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/xor.png)  

首先，将输入值分别进行按位与`&`、按位或`|`计算，获得`0 0 0 1`、`0 1 1 1`。  

然后，关键点来了，将按位与的结果取反`!`！！！，获得`1 1 1 0`。  

最后将取反的结果与第一步中按位或的结果再进行按位与操作`&`，获得异或的结果： `0 1 1 0`。  

`bingo!`  


最后，推荐一门课程，最近天天都在地铁上看，已经第二遍了~  

`A new layer of abstraction~`

[计算机科学速成课[40集全/精校] - Crash Course Computer Science](https://www.bilibili.com/video/av21376839?p=3){:target="_blank"}

