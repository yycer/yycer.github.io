---
layout: post
title: 小灰 - 金矿问题  
category: algorithm
tags: [algorithm]
excerpt: Gold mining
---

## 问题描述  

假设金矿数量为`n`，工人数量为`w`，金矿的含金量设为数组`g[]`，  

金矿所需开采人数为数组`p[]`，求`n`个金矿，`w`个工人的最优收益是多少？  


举个例子：  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/gold_mining_problem.png)  


## 解法1  

这道题很有意思，让我更加深入的理解`DP`的三要素：  

`最优子结构`、`状态转移方程`和`边界`。  

什么是`最优子结构`？  

我觉得它与`拆分子问题思想`，还有`状态转移方程`紧密相关。  

举个例子：  

假设，你有`3座金矿`、`5个工人`，并且此时的最大利润已经确定：  

`maxProfit = F(3, 5)`   


接着，又发现了第`4`座金矿，此时，你有两个选择：  

第一个，人数不够，第`4`座金矿没法挖，此时你的最大利润还是`F(3, 5)`  

第二个，人数足够，如：派`x`个人去挖第`4`座金矿，此时，又分为两种场景：  

我派这`x`个人去挖第`4`座金矿是否合算？  

因为这`x`个人可能是从其他矿区抽出来的，如果他们去其他矿区挖，是否会带来更高的利润？  

假设第四座金矿的利润为： `profit(4)`， 我们可以得到如下式子：  



`F(4, 5) = Max(F(3, 5), F(3, 5 - x) + profit(4))`  


我们再来解释下`边界`与`状态转移方程`：  


我们将金矿数量设为`n`，工人数量为`w`，金矿的含金量设为数组`g[]`，金矿所需开采人数为数组`p[]`。  

如果没有剩余工人、或者没有金矿，那么利润肯定为`0`：  

`F(n, w) = 0, n = 0 || w == 0`  

如果没有剩余工人，但是有多余的金矿，那么也只能维持之前的利润：  

`F(n, w) = F(n - 1, w), n >= 1 && w < p[n - 1]`  

其他情况，你有两种选择，要么挖当前金矿，要么不挖：  

`F(n, w) = Max(F(n - 1, w), F(n - 1, w - p[n - 1]) + g[n - 1]), n >= 1 && w >= p[n - 1]`   


来看下示意图：  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/gold_mining_key.png)  


来看下实现：  


``` java
private static void getBestGoldMining() {

    int   n = 5;  // Number of gold mines.
    int   w = 10; // Number of workers.
    int[] p = { 5,   5,   3,   4,   3 }; // Number of workers required for gold mining.
    int[] g = {400, 500, 200, 300, 350}; // Gold reverses.

    int[][] dp = new int[n + 1][w + 1];
    for (int i = 1; i <= n; i++){
        for (int j = 1; j <= w; j++){
            // Don't have enough workers.
            if (j < p[i - 1]){
                dp[i][j] = dp[i - 1][j];
            } else {
                // State transition equation.
                dp[i][j] = Math.max(dp[i - 1][j], dp[i - 1][j - p[i - 1]] + g[i - 1]);
            }
        }
    }

    System.out.println(dp[n][w]);
}
```

压缩成一维数组：  

注意，内层循环需要`从右往左`遍历，避免使用重复值。  

``` java
private static void getBestGoldMiningOptimize() {

    int   n = 5;  // Number of gold mines.
    int   w = 10; // Number of workers.
    int[] p = { 5,   5,   3,   4,   3 }; // Number of workers required for gold mining.
    int[] g = {400, 500, 200, 300, 350}; // Gold reverses.

    int[] dp = new int[w + 1];
    for (int i = 1; i <= n; i++){
        for (int j = w; j >= p[i - 1]; j--){
            // State transition equation.
            dp[j] = Math.max(dp[j], dp[j - p[i - 1]] + g[i - 1]);
        }
    }

    System.out.println(dp[w]);
}
```

`Enjoy it ! `