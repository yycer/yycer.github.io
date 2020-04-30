---
layout: post
title: 算法 - 大O复杂度表示法
category: algorithm
tags: [algorithm]
excerpt: Big O
---

之前也看过一些讲解`大O复杂度`的文章，但一直模棱两可。  
直到我看到了这两篇文章：[复杂度分析（上）和（下）](https://time.geekbang.org/column/article/40036){:target="_blank"}， 才有了更深一步的理解。  


首先来看下公式：  

``` java
T(n) = O(f(n))
```

分别代表什么意思？  


`T(n)`：代表所有代码的执行时间。  
`f(n)`：代表每行代码的执行次数。  


举个例子：  

``` java
public void method(int n){
    int i = 0;   // line 1
    int j = 0;   // line 2
    for (; i < n; i++){            // line 3
        System.out.println(i);     // line 4
        for (; j < n; i++){        // line 5
            System.out.println(j); // line 6
        }
    }
}
```

我们来分析一下：  

`line 1和2`分别执行`1`次。  
`line 3和4`分别执行`n`次。  
`line 5和6`分别执行`n^2`次。   
 
所以`f(n) = 2n^2 + 2n + 2`。我们假设每行代码执行的时间相同，都是`unit_time`。  

那么`T(n) = (2n^2 + 2n + 2) * unit_time`

可以得出如下结论：所有代码的执行时间`T(n)`和每行代码的执行次数`f(n)`成`正比`。  

这就是大O的由来，可以表示成： `T(n) = O(2n^2 + 2n + 2)`  

> 大 O 时间复杂度实际上并不具体表示代码真正的执行时间，而是表示代码执行时间随数据规模增长的变化趋势，所以，也叫作渐进时间复杂度（asymptotic time complexity），简称时间复杂度。  

> 当 n 很大时，你可以把它想象成 10000、100000。而公式中的`系数`、`低阶`、`常量`三部分并不左右增长趋势，所以都可以忽略。我们只需要记录一个最大量级就可以了。  

因此，我们可以简化成`T(n) = O(n^2)`  