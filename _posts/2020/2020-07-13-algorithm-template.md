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


## 归并排序  

分为以下三步：  

第一步，确定分界点，注意是中间的`索引`，而不是对应的值。  

第二步，递归排序左右两边。  

第三步，合二为一。  

来看下实现：  

``` java
private static void mergeSort(int[] arr, int l, int r){
    if (l >= r) return;
    
    int mid = l + r >>> 1;
    mergeSort(arr, l, mid);
    mergeSort(arr, mid + 1, r);
    
    // 开始归并
    int k = 0, i = l, j = mid + 1;
    while (i <= mid && j <= r){
        if (arr[i] <= arr[j]) tmp[k++] = arr[i++];
        else tmp[k++] = arr[j++];
    }

    // 扫尾
    while (i <= mid) tmp[k++] = arr[i++];
    while (j <= r)   tmp[k++] = arr[j++];

    // 物归原主
    for (int m = l, n = 0; m <= r; m++, n++){
        arr[m] = tmp[n];
    }
}
```