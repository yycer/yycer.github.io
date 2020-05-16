---
layout: post
title: 算法与数据结构[排序] - 选择排序
category: algorithm
tags: [algorithm]
excerpt: Algorithm and Data structure - Selection Sort
---

本文介绍下选择排序，它的核心思想是：`从无序区域中选择最小的元素，并将其放至左侧。`  

它的时间复杂度也是`O(N^2)`，那么它与冒泡排序的区别是什么？  

> 减少了交互次数。  

来下看示意图：  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/selection_sort.png)  


最后来看下代码实现：  

``` java
public class SelectionSortTest {

    public static void main(String[] args) {
        int[] arr = {5, 3, 5, 2, 4};
        selectionSort(arr);
        System.out.println(Arrays.toString(arr));
    }

    private static void selectionSort(int[] arr) {
        for (int i = 0; i < arr.length - 1; i++){
            int min = i;
            for (int j = i + 1; j < arr.length; j++){
                if (arr[j] < arr[min]){
                    min = j;
                }
            }
            if (min != i){
                Utils.swap(arr, i, min);
            }
        }
    }
}

public class Utils {

    public static void swap(int[] arr, int i, int j){
        int tmp = arr[i];
        arr[i] = arr[j];
        arr[j] = tmp;
    }
}
```
