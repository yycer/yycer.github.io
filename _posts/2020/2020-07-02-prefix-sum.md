---
layout: post
title: 算法与数据结构 - 前缀和总结
category: algorithm
tags: [algorithm]
excerpt: Algorithm and Data structure - Prefix Sum  
---


- 为什么要用`前缀和`思想？  

它能以`O(n)`的时间复杂度处理`连续子数组`问题。  

- 什么是`前缀和`数组？  

来看下示意图：  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/leetcode_560_presum_1.png)  

很简单，所谓`前缀和`数组，就是依次遍历原始数组，累加当前元素与之前的所有元素之和。  

注意，我们需要多一个首元素`0`，便于后续的计算。  

- `前缀和`数组的有什么用处？  

举个例子，如何计算`nums[0]`到`nums[1]`区间内的和？  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/leetcode_560_presum_2.png)  

有没有发现： `sum of range[0, 1] = preSum[2] - preSum[0] = 3 - 0 = 3`    


再来，如何计算`nums[0]`到`nums[3]`区间内的和？   

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/leetcode_560_presum_3.png)  

老规矩： `sum of range[0, 3] = preSum[4] - preSum[0] = 5 - 0 = 5`    


继续，如何计算`nums[2]`到`nums[3]`区间内的和？   

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/leetcode_560_presum_4.png)  

来简单推导一下：  

``` java
sum of range[2, 3]
= sum of range[0, 3] - sum of range[0, 1]
= (preSum[4] - preSum[0]) - (preSum[2] - preSum[0])
= preSum[4] - preSum[2]
= 5 - 3
= 2
```


## 实战        

- 1、[[560] Subarray Sum Equals K](http://yaoyichen.cn/algorithm/2020/06/30/leetcode-560.html){:target="_blank"}  
- 2、[[1480] Running Sum of 1d Array](http://yaoyichen.cn/algorithm/2020/06/30/leetcode-1480.html){:target="_blank"}  
- 3、[[1413] Minimum Value to Get Positive Step by Step Sum](http://yaoyichen.cn/algorithm/2020/06/30/leetcode-1413.html){:target="_blank"}  
- 4、[[1310] XOR Queries of a Subarray](http://yaoyichen.cn/algorithm/2020/07/01/leetcode-1310.html){:target="_blank"}  
- 5、[[930] Binary Subarrays With Sum](http://yaoyichen.cn/algorithm/2020/07/01/leetcode-930.html){:target="_blank"}  
- 6、[[974] Subarray Sums Divisible by K](http://yaoyichen.cn/algorithm/2020/06/30/leetcode-974.html){:target="_blank"}  
- 7、[[303] Range Sum Query - Immutable](http://yaoyichen.cn/algorithm/2020/06/30/leetcode-303.html){:target="_blank"}  
- 8、[[307] Range Sum Query - Mutable](http://yaoyichen.cn/algorithm/2020/06/30/leetcode-307.html){:target="_blank"}  
- 9、[[523] Continuous Subarray Sum](http://yaoyichen.cn/algorithm/2020/06/30/leetcode-523.html){:target="_blank"}  
- 10、[[437] Path Sum III](http://yaoyichen.cn/algorithm/2020/04/06/leetcode-437.html){:target="_blank"}  
- 11、[[724] Find Pivot Index](http://yaoyichen.cn/algorithm/2020/07/01/leetcode-724.html){:target="_blank"}  





