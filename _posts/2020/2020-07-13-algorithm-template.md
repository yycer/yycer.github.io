---
layout: post
title: 算法模板  
category: algorithm
tags: [algorithm]
excerpt: Algorithm Template  
---

本文梳理下常用的算法模板：   


## 快速排序  

主要分为以下三步：  

第一，确定分界点`target`，一般选择中间位置的值。  

第二，调整区间，将小于等于`target`的值放在左边，否则放在右边。  

第三，递归处理左右两段。   


来看下模板代码：  

``` java
private void quickSort(int[] arr, int l, int r){
    if (l >= r) return;
    
    int target = arr[(l + r) >>> 1];
    int i = l - 1;
    int j = r + 1;
    
    while (i < j){
        while (arr[++i] < target);
        while (arr[--j] > target);
        if (i < j) swap(arr, i, j);
    }
    
    quickSort(arr, l, j);
    quickSort(arr, j + 1, r);
}
```
