---
layout: post
title: 算法与数据结构[排序] - 快速排序
category: algorithm
tags: [algorithm]
excerpt: Algorithm and Data structure - Quick Sort
---

本文让我们介绍下快速排序，此刻我们应该感到`Amazing`，因为这是我们讲的第一个时间复杂度为`NO(logN)`的排序算法！  

核心思想如下：  


``` java
int pivotIndex = getPivotIndex();
quickSort(arr, lo, pivotIndex - 1);
quickSort(arr, pivotIndex + 1, hi);
```

关键在于如何选择并获取`pivotIndex`，本文主要介绍两种方式：  

- 双边循环法  
- 单边循环法  

## 双边循环法  

我们暂定首个元素为`pivot`，  
`从右到左`找到一个小于等于`pivot`的元素，并`从左到右`找到第一个大于`pivot`的元素，  
然后彼此交换，接着继续遍历，直至两者相交，  
最后首个元素与交点元素互换，并返回交点元素的下标！  

先来看下示意图：  


![](https://yyc-images.oss-cn-beijing.aliyuncs.com/quick_sort_double_pointer_round1.png)  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/quick_sort_double_pointer_round2.png)  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/quick_sort_double_pointer_round3.png)  


再来看下代码实现：  

``` java

public static void main(String[] args) {
    int[] nums = {5, 3, 8, 4, 1, 7, 9};
    quickSortUsingDoublePointer(nums);
    System.out.println(Arrays.toString(nums));
    // [1, 3, 4, 5, 7, 8, 9]
}

private static void quickSortUsingDoublePointer(int[] nums) {
    quickSortUsingDP(nums, 0, nums.length - 1);
}

private static void quickSortUsingDP(int[] nums, int lo, int hi) {
    if (lo >= hi){
        return;
    }

    int pivotIndex = getPivotIndexUsingDP(nums, lo, hi);
    quickSortUsingDP(nums, lo, pivotIndex - 1);
    quickSortUsingDP(nums, pivotIndex + 1, hi);
}

private static int getPivotIndexUsingDP(int[] nums, int lo, int hi) {
    int pivot = nums[lo];
    int start = lo;
    int end   = hi;

    while (start != end){
        while (start < end && nums[end] > pivot){
            end--;
        }
        while (start < end && nums[start] <= pivot){
            start++;
        }
        if (start < end){
            Utils.swap(nums, start, end);
        }
    }

    nums[lo]    = nums[start];
    nums[start] = pivot;

    return start;
}

public class Utils {

    public static void swap(int[] arr, int i, int j){
        int tmp = arr[i];
        arr[i] = arr[j];
        arr[j] = tmp;
    }
}
```


## 单边循环法  

再来介绍下单边循环法：  

同样，暂定第一个首元素为`pivot`，并创建`mark`变量，其值为首元素下标。  

接着，从第二个元素开始向右变量，若当前元素大于`pivot`，则跳过。  

否则，`mark++`，同时互换当前元素与`arr[mark]`。  

最后，互换首元素与`arr[mark]`，并返回`mark`。  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/quick_sort_single_pointer_round1.png)  


来看下代码实现：  

``` java
public static void main(String[] args) {
    int[] nums = {4, 3, 5, 2, 4};
    quickSortUsingSinglePointer(nums);
    System.out.println(Arrays.toString(nums));
    // [2, 3, 4, 4, 5]
}

private static void quickSortUsingSinglePointer(int[] nums) {
    quickSortUsingSP(nums, 0, nums.length - 1);
}


private static void quickSortUsingSP(int[] nums, int lo, int hi) {
    if (lo >= hi) return;

    int pivotIndex = getPivotIndexUsingSP(nums, lo, hi);
    quickSortUsingSP(nums, lo, pivotIndex - 1);
    quickSortUsingSP(nums, pivotIndex + 1, hi);
}

private static int getPivotIndexUsingSP(int[] nums, int lo, int hi) {
    int pivot = nums[lo];
    int mark  = lo;

    for (int i = lo + 1; i <= hi; i++){
        if (nums[i] < pivot){
            mark++;
            Utils.swap(nums, i, mark);
        }
    }

    nums[lo]   = nums[mark];
    nums[mark] = pivot;

    return mark;
}
```

## Reference  
- `《漫画算法 - 小灰的算法之旅》 - 4.3 什么是快速排序`  