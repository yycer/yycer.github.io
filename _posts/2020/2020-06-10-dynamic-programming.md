---
layout: post
title: 算法与数据结构 - 动态规划总结
category: algorithm
tags: [algorithm]
excerpt: Algorithm and Data structure - Dynamic Programming
---


本文来总结下动态规划思想。  

- 它的由来和特点是什么？  

> 动态规划思想一般使用穷举、数据暂存的方式求最值问题。  

举个例子 [[322] Coin Change](http://yaoyichen.cn/algorithm/2020/06/07/leetcode-322.html){:target="_blank"}  

给你三种面值、无限提供的硬币： `coins = [1, 2 ,5]`，请计算凑出金额`amount = 6`的最少硬币数量。  

人脑很聪明，一下就能反应过来，不就是`2枚硬币`么，`6 = 5 + 1`。  

但是随着`coins`和`amount`规模的增加，人脑就计算不过来了。  

此时，我们就要利用计算机的计算能力，使用穷举的方式列出所有可能性，然后寻找其中的最优解。  
![](https://yyc-images.oss-cn-beijing.aliyuncs.com/leetcode_322_using_dp_common.png)  


当然，在整个计算过程中无法避免重复计算的问题，因此，  

我们可以使用某些数据结构暂存部分数据，避免重复计算，比如：临时变量或`一/二/多`维数组。  


- 它的核心思想是什么？  

> 穷举所有可能性，找出数据之间的规律，并暂存部分结果，最后获得最优解。  


- 技巧总结  

首先，你需要找到一个符合题目要求、足够精简的样例数据。  

然后，从二(多)个维度穷举所有的可能性，从中找到隐藏在数据中的规律`(公式)`。  

比如，如果两个字符相等时，应该怎么办？不等，又该怎么办？  

使用前`i`种面值的硬币，凑齐`j`元的方式有哪几种？每种面值的硬币是否可以无限使用？等等。  

此时，常规的`DP`题目，你的主干部分应该写得差不多了。  

接下来，需要结合一些边界`Test Case`考虑初始值与边界条件。  

比如： `dp[0][0]`为`0`还是`1`，为什么？第一行的其他元素为`0`还是`1`？为什么？  


再来，我们可以对上面的解法做一些优化，比如：将二维数组压缩成一维的，减少内存占用和代码复杂度。  

为什么可以压缩成一维数组？什么情况下可以这么做？  

你会发现一个规律，`dp[i][j]`大部分情况下，会和它的`左`、`上`和`左上`元素有某种关系。  

那么，我用一维数组就足够了。  



## 实战  

纸上得来终觉浅，绝知此事要躬行。  

- 1、[[70] Climbing Stairs](http://yaoyichen.cn/algorithm/2020/04/05/leetcode-70.html){:target="_blank"}  
- 2、[[746] Min Cost Climbing Stairs](http://yaoyichen.cn/algorithm/2020/05/30/leetcode-746.html){:target="_blank"}  
- 3、[[198] House Robber](http://yaoyichen.cn/algorithm/2020/04/08/leetcode-198.html){:target="_blank"}  
- 4、[[213] House Robber II](http://yaoyichen.cn/algorithm/2020/05/30/leetcode-213.html){:target="_blank"}  
- 5、[[509] Fibonacci Number](http://yaoyichen.cn/algorithm/2020/04/05/leetcode-509.htmls){:target="_blank"}  
- 6、[[64] Minimum Path Sum](http://yaoyichen.cn/algorithm/2020/05/30/leetcode-64.html){:target="_blank"}  
- 7、[[62] Unique Paths](http://yaoyichen.cn/algorithm/2020/02/24/leetcode-62.html){:target="_blank"}  
- 8、[[413] Arithmetic Slices](http://yaoyichen.cn/algorithm/2020/06/05/leetcode-413.html){:target="_blank"}  
- *9、[[343] Integer Break](http://yaoyichen.cn/algorithm/2020/06/05/leetcode-343.html){:target="_blank"}  
- 10、[[279] Perfect Squares](http://yaoyichen.cn/algorithm/2020/06/05/leetcode-279.html){:target="_blank"}
- 11、[[300] Longest Increasing Subsequence](http://yaoyichen.cn/algorithm/2020/06/05/leetcode-300.html){:target="_blank"}  
- 12、[[376] Wiggle Subsequence](http://yaoyichen.cn/algorithm/2020/06/05/leetcode-376.html){:target="_blank"}  
- 13、[[1143] Longest Common Subsequence](http://yaoyichen.cn/algorithm/2020/06/05/leetcode-1143.html){:target="_blank"}  
- *14、[[416] Partition Equal Subset Sum](http://yaoyichen.cn/algorithm/2020/06/05/leetcode-416.html){:target="_blank"}
- 16、[[516] Longest Palindromic Subsequence](http://yaoyichen.cn/algorithm/2020/06/06/leetcode-516.html){:target="_blank"}  
- 18、[[322] Coin Change](http://yaoyichen.cn/algorithm/2020/06/07/leetcode-322.html){:target="_blank"}    
- 19、[[518] Coin Change 2](http://yaoyichen.cn/algorithm/2020/06/07/leetcode-518.html){:target="_blank"}    
- 20、[[139] Word Break](http://yaoyichen.cn/algorithm/2020/06/07/leetcode-139.html){:target="_blank"}  
- *21、[[377] Combination Sum IV](http://yaoyichen.cn/algorithm/2020/06/07/leetcode-377.html){:target="_blank"}  
- 22、[[121] Best Time to Buy and Sell Stock](http://yaoyichen.cn/algorithm/2020/06/08/leetcode-121.html){:target="_blank"}    
- 23、[[122] Best Time to Buy and Sell Stock II](http://yaoyichen.cn/algorithm/2020/06/08/leetcode-122.html){:target="_blank"}    
- 24、[[123] Best Time to Buy and Sell Stock III](http://yaoyichen.cn/algorithm/2020/06/08/leetcode-123.html){:target="_blank"}  
- 25、[[309] Best Time to Buy and Sell Stock with Cooldown](http://yaoyichen.cn/algorithm/2020/06/08/leetcode-309.html){:target="_blank"}    
- 26、[[714] Best Time to Buy and Sell Stock with Transaction Fee](http://yaoyichen.cn/algorithm/2020/06/08/leetcode-714.html){:target="_blank"}    
- 27、[[583] Delete Operation for Two Strings](http://yaoyichen.cn/algorithm/2020/06/08/leetcode-583.html){:target="_blank"}  
- 28、[[72] Edit Distance](http://yaoyichen.cn/algorithm/2020/06/08/leetcode-72.html){:target="_blank"}    
- *29、[[650] 2 Keys Keyboard](http://yaoyichen.cn/algorithm/2020/06/09/leetcode-650.html){:target="_blank"}    
- *30、[[337] House Robber III](http://yaoyichen.cn/algorithm/2020/06/11/leetcode-337.html){:target="_blank"}   
- *31、[[96] Unique Binary Search Trees](http://yaoyichen.cn/algorithm/2020/06/18/leetcode-96.html){:target="_blank"}   
- 32、[[53] Maximum Subarray](http://yaoyichen.cn/algorithm/2020/02/19/leetcode-53.html){:target="_blank"}   
- 33、[[338] Counting Bits](http://yaoyichen.cn/algorithm/2020/03/21/leetcode-338.html){:target="_blank"}   
- 34、[[264] Ugly Number II](http://yaoyichen.cn/algorithm/2020/06/25/leetcode-264.html){:target="_blank"}   
- *35、[[647] Palindromic Substrings](http://yaoyichen.cn/algorithm/2020/06/26/leetcode-647.html){:target="_blank"}   
- 36、[[5] Longest Palindromic Substring](http://yaoyichen.cn/algorithm/2020/06/26/leetcode-5.html){:target="_blank"}   
- 37、[小灰 - 金矿问题](http://yaoyichen.cn/algorithm/2020/07/03/gold-mining.html){:target="_blank"}   


## `0-1 Knapsack`  

- 1、[[416] Partition Equal Subset Sum](http://yaoyichen.cn/algorithm/2020/06/05/leetcode-416.html){:target="_blank"}  
- 2、[[474] Ones and Zeroes](http://yaoyichen.cn/algorithm/2020/06/06/leetcode-474.html){:target="_blank"}  
- 3、[[494] Target Sum](http://yaoyichen.cn/algorithm/2020/06/06/leetcode-494.html){:target="_blank"}  
- 4、[[1049] Last Stone Weight II](http://yaoyichen.cn/algorithm/2020/09/14/leetcode-1049.html){:target="_blank"}  




