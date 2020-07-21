---
layout: post
title: 算法与数据结构 - 位运算总结
category: algorithm
tags: [algorithm]
excerpt: Algorithm and Data structure - Bit Manipulation
---


本文总结下位运算的常见题目。  

常见的位运算操作有：`与[&]`、`或[|]`、`异或[^]`操作。  

比如：如何判断一个正整数`n`是否为奇数？  

``` java
(n & 1) == 1;
```

再如：如何将一个整数`n`除以`2`？  

``` java
n >>>= 1;
```


## 单个数字系列    

- 1、[[136] Single Number](http://yaoyichen.cn/algorithm/2020/03/13/leetcode-136.html){:target="_blank"}  
- 2、[[137] Single Number II](http://yaoyichen.cn/algorithm/2020/06/24/leetcode-137.html){:target="_blank"}  
- 3、[[260] Single Number III](http://yaoyichen.cn/algorithm/2020/03/22/leetcode-260.html){:target="_blank"}  


## 幂系列    

- 1、[[231] Power of Two](http://yaoyichen.cn/algorithm/2020/03/21/leetcode-231.html){:target="_blank"}  
- *2、[[326] Power of Three](http://yaoyichen.cn/algorithm/2020/06/24/leetcode-326.html){:target="_blank"}  
- 3、[[342] Power of Four](http://yaoyichen.cn/algorithm/2020/03/21/leetcode-342.html){:target="_blank"}  


## 其他      

- 1、[[461] Hamming Distance](http://yaoyichen.cn/algorithm/2020/06/23/leetcode-461.html){:target="_blank"}  
- *2、[[191] Number of 1 Bits](http://yaoyichen.cn/algorithm/2020/06/23/leetcode-191.html){:target="_blank"}  
- 3、[[268] Missing Number](http://yaoyichen.cn/algorithm/2020/02/17/leetcode-268.html){:target="_blank"}  
- **4、[[693] Binary Number with Alternating Bits](http://yaoyichen.cn/algorithm/2020/03/21/leetcode-693.html){:target="_blank"}  
- 5、[[476] Number Complement](http://yaoyichen.cn/algorithm/2020/03/19/leetcode-476.html){:target="_blank"}  
- *6、[[371] Sum of Two Integers](http://yaoyichen.cn/algorithm/2020/06/24/leetcode-371.html){:target="_blank"}  
- *7、[[338] Counting Bits](http://yaoyichen.cn/algorithm/2020/03/21/leetcode-338.html){:target="_blank"}  