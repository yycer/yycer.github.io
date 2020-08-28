---
layout: post
title: 算法与数据结构 - 二分查找总结
category: algorithm
tags: [algorithm]
excerpt: Algorithm and Data structure - Binary Search
---


本文来总结下二分查找思想。  

它的核心思想： `性质的边界`   


## 模板1  

先来看下示意图：  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/binary_search_module_1.png)  
那么我们要求的答案有什么特点？   

> 第一次满足某个性质  

分为两种情况，当`mid`满足了这个性质：  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/binary_search_module_1_right.png)  

说明什么？  

> 答案在左边，且mid可能就是答案。  

否则，答案在右边。  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/binary_search_module_1_left.png)  

``` java
int bsearch_1(int l, int r)
{
    while (l < r)
    {
        int mid = l + r >> 1;
        if (check(mid)) r = mid;
        else l = mid + 1;
    }
    return l;
}
```


## 模板2  
  

老规矩：  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/binary_search_module_2.png)  

我们要找的答案有什么特点？  

> 最后一次满足某个性质。   


也分为两种情况，如果`mid`满足这个性质：  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/binary_search_module_2_left.png)  


说明什么？  

> 答案在右边，或者mid就是答案。  


否则，答案在左边。  



![](https://yyc-images.oss-cn-beijing.aliyuncs.com/binary_search_module_2_right.png)  


``` java
int bsearch_2(int l, int r)
{
    while (l < r)
    {
        int mid = l + r + 1 >> 1;
        if (check(mid)) l = mid;
        else r = mid - 1;
    }
    return l;
}
```

注意`+1`的目的，是防止`向下取整`，最终导致死循环。  



## 实战  

- *1、[[69] Sqrt(x)](http://yaoyichen.cn/algorithm/2020/05/17/leetcode-69.html){:target="_blank"}  
- 2、[[367] Valid Perfect Square](http://yaoyichen.cn/algorithm/2020/06/25/leetcode-367.html){:target="_blank"}  
- 3、[[540] Single Element in a Sorted Array](http://yaoyichen.cn/algorithm/2020/05/18/leetcode-540.html){:target="_blank"}  
- 4、[[278] First Bad Version](http://yaoyichen.cn/algorithm/2020/05/20/leetcode-278.html){:target="_blank"}  
- 5、[[153] Find Minimum in Rotated Sorted Array](http://yaoyichen.cn/algorithm/2020/05/20/leetcode-153.html){:target="_blank"}  
- 6、[[154] Find Minimum in Rotated Sorted Array II](http://yaoyichen.cn/algorithm/2020/07/15/leetcode-154.html){:target="_blank"}  
- 7、[[33] Search in Rotated Sorted Array](http://yaoyichen.cn/algorithm/2020/07/15/leetcode-33.html){:target="_blank"}  
- 8、[[81] Search in Rotated Sorted Array II](http://yaoyichen.cn/algorithm/2020/07/15/leetcode-81.html){:target="_blank"}  
- 9、[[34] Find First and Last Position of Element in Sorted Array](http://yaoyichen.cn/algorithm/2020/05/22/leetcode-34.html){:target="_blank"}  
- 10、[[74] Search a 2D Matrix](http://yaoyichen.cn/algorithm/2020/07/02/leetcode-74.html){:target="_blank"}  
- 11、[[162] Find Peak Element](http://yaoyichen.cn/algorithm/2020/07/15/leetcode-162.html){:target="_blank"}  
- 12、[[852] Peak Index in a Mountain Array](http://yaoyichen.cn/algorithm/2020/07/15/leetcode-852.html){:target="_blank"}  
- 13、[[35] Search Insert Position](http://yaoyichen.cn/algorithm/2020/07/15/leetcode-35.html){:target="_blank"}  
- 14、[[668] Kth Smallest Number in Multiplication Table](http://yaoyichen.cn/algorithm/2020/08/28/leetcode-668.html){:target="_blank"}  


