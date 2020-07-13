---
layout: post
title: 算法与数据结构[排序] - 归并排序
category: algorithm
tags: [algorithm]
excerpt: Algorithm and Data structure - Merge Sort
---


## Update at 2020_0713  

总结一下归并排序的模板，`From Y总`  

先梳理下思路，主要分为三步：  

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



我们再来介绍下另一种时间复杂度为`NO(logN)`的排序算法：归并排序。  


那么它与快速排序的区别是什么？  

快速排序的核心是使用`pivot`进行`分治`，而归并排序也用到了分治思想，不过当它`治`的时候，会使用一些额外的内存空间。  

老规矩，先来看下示意图：  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/merge_sort.png)  


再来看下实现：  

``` java
public static void main(String[] args) {
    int[] arr = {4, 3, 5, 2, 6, 4};
    mergeSort(arr);
    System.out.println(Arrays.toString(arr));
    // [2, 3, 4, 4, 5]
}

private static void mergeSort(int[] arr) {
    sep(arr, 0, arr.length - 1);
}

private static void sep(int[] arr, int l, int r) {
    if (l >= r) {
        return;
    }

    int mid = l + ((r - l) >> 1);
    sep(arr, l, mid);
    sep(arr, mid + 1, r);
    merge(arr, l, mid + 1, r);
}

private static void merge(int[] arr, int l, int mid, int r) {
    int[] left  = new int[mid - l];
    int[] right = new int[r - mid + 1];

    // Fill left and right.
    for (int i = l; i < mid; i++){
        left[i - l] = arr[i];
    }

    for (int j = mid; j <= r; j++){
        right[j - mid] = arr[j];
    }

    // Adjust arr using left and right.
    int i = 0, j = 0, k = l;
    while (i < left.length && j < right.length){
        if (left[i] < right[j]){
            arr[k++] = left[i++];
        } else {
            arr[k++] = right[j++];
        }
    }

    while (i < left.length){
        arr[k++] = left[i++];
    }

    while (j < right.length){
        arr[k++] = right[j++];
    }
}
```