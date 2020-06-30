---
layout: post
title: 算法与数据结构 - 二分查找总结
category: algorithm
tags: [algorithm]
excerpt: Algorithm and Data structure - Binary Search
---


本文来总结下二分查找思想。  

它的由来和特点是什么？  

> 针对有序数据集合，以log(n)的时间复杂度高效查询某一元素。  

它的核心思想是什么？  

> 每次都与区间内的中间元素对比，将待查找的区间缩小为一半，直到找到查找的元素，或者区间被缩小为0。  

它的易出错点有哪些？  

> lo和hi的取值、更新和返回、循环退出条件。  

它有哪些应用场景？  

> 底层依赖有序数组。对于较小规模的数据查找，可以直接顺序遍历，二分查找的优势并不明显。
二分查找更适合处理静态数据，也就是没有频繁的数据插入、删除操作，做到一次排序，多次查询。
  

## 实战  

- [[69] Sqrt(x)](http://yaoyichen.cn/algorithm/2020/05/17/leetcode-69.html){:target="_blank"}  
- [[744] Find Smallest Letter Greater Than Target](http://yaoyichen.cn/algorithm/2020/05/18/leetcode-744.html){:target="_blank"}  
- [[540] Single Element in a Sorted Array](http://yaoyichen.cn/algorithm/2020/05/18/leetcode-540.html){:target="_blank"}  
- [[278] First Bad Version](http://yaoyichen.cn/algorithm/2020/05/20/leetcode-278.html){:target="_blank"}  
- [[153] Find Minimum in Rotated Sorted Array](http://yaoyichen.cn/algorithm/2020/05/20/leetcode-153.html){:target="_blank"}  
- [[34] Find First and Last Position of Element in Sorted Array](http://yaoyichen.cn/algorithm/2020/05/22/leetcode-34.html){:target="_blank"}  
- [[167] Two Sum II - Input array is sorted](http://yaoyichen.cn/algorithm/2020/05/23/leetcode-167.html){:target="_blank"}  
