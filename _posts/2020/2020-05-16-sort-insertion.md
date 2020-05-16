---
layout: post
title: 算法与数据结构[排序] - 插入排序
category: algorithm
tags: [algorithm]
excerpt: Algorithm and Data structure - Insertion Sort
---

本文介绍下插入排序，它的时间复杂度与`冒泡`、`选择`排序一样，都是`O(N^2)`。  

它的核心思想是：`从第二个元素开始，将当前元素插入有序区域。`可以想象下抓牌过程。    


来看下示意图：  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/insertion_sort.png)  


再来看下代码实现：  


``` java
public class InsertionSortTest {

    public static void main(String[] args) {
        int[] arr = {5, 3, 5, 2, 4};
        insertionSort(arr);
        System.out.println(Arrays.toString(arr));
    }

    private static void insertionSort(int[] arr) {
        for (int i = 1; i < arr.length; i++){
            for (int j = i; j > 0; j--){
                if (arr[j] < arr[j - 1]){
                    Utils.swap(arr, j, j - 1);
                }
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