---
layout: post
title: 算法与数据结构[排序] - 冒泡排序
category: algorithm
tags: [algorithm]
excerpt: Algorithm and Data structure - Bubble Sort
---

本文介绍下冒泡排序，它的时间复杂度为`O(N^2)`，作为一种入门排序算法，还是值得一学的。  

当然，我还总结了几种特殊的场景，比如：如何避免冗余的比较？如何高效地处理原始数组内已部分排序的情况？  


## 普通冒泡排序  

先说下冒泡排序的核心思想： `将当前元素与前一个元素进行比较，若小于，则互换位置，否则，继续遍历下一个元素。`  

可以思考下，为什么不是小于等于时互换位置？  

来看下示意图：  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/bubble_sort_common_round1.png)  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/bubble_sort_common_round2.png)  


![](https://yyc-images.oss-cn-beijing.aliyuncs.com/bubble_sort_common_round3.png)  


![](https://yyc-images.oss-cn-beijing.aliyuncs.com/bubble_sort_common_round4.png)  


可以看到为了保证数组的`稳定性`，所以当两个元素的值相等时，才没有互换位置。  

那什么是数组稳定性？  

值相等的元素并不会打乱原本的顺序。  


再来看下代码实现：  

``` java
public static void playBubble(int[] arr){
    int compareCount = 0;
    int len = arr.length;
    for (int i = len - 1; i > 0; i--){
        for (int j = 1; j <= i; j++){
            compareCount++;
            if (arr[j] < arr[j - 1]){
                Utils.swap(arr, j, j- 1);
            }
        }
    }
    System.out.println(compareCount);
}

public class Utils {

    public static void swap(int[] arr, int i, int j){
        int tmp = arr[i];
        arr[i] = arr[j];
        arr[j] = tmp;
    }
}
```


## 有序标识思想  

现在，让我们开始进行优化。  

你有没有觉得，上面的`4.1`步骤很冗余？因为执行完`3.2`后数组已经完成排序。  

让我们添加一个已完成排序的标识：  

``` java
private static void playBubbleJudgeSorted(int[] arr) {
    int len = arr.length;
    int compareCount = 0;
    for (int i = len - 1; i > 0; i--){
        boolean isSorted = true;
        for (int j = 1; j <= i; j++){
            compareCount++;
            if (arr[j] < arr[j - 1]){
                Utils.swap(arr, j, j - 1);
                isSorted = false;
            }
        }
        if (isSorted){
            break;
        }
    }
    System.out.println(compareCount);
}
```

来看下示意图：  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/bubble_sort_is_sorted_round1.png)  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/bubble_sort_is_sorted_round2.png)  

我们可以做个比较，调整前比较次数为`6*5/2=15`，调整后比较次数为`9`！  


## 边界思想    

再来考虑下另一种特殊场景，如果原始数组中前半部分已有序，还有没有更高效的解法？  

此时，就可以使用`边界法`：  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/bubble_sort_border_round1.png)  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/bubble_sort_border_round2.png)  


``` java
private static void playBubbleUsingBorder(int[] arr) {

    int len = arr.length;
    int compareCount      = 0;
    int lastExchangeIndex = 0;
    for (int border = len - 1; border > 0;){
       boolean isSorted = true;
       for (int j = 1; j <= border; j++){
           compareCount++;
           if (arr[j] < arr[j - 1]){
               Utils.swap(arr, j, j - 1);
               isSorted = false;
               lastExchangeIndex = j - 1;
           }
       }
       border = lastExchangeIndex;
       if (isSorted){
           break;
       }
    }
    System.out.println(compareCount);
}
```

## 鸡尾酒思想  

再来看最后一种更为极端的场景，大多数前半部分元素均已有序：  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/bubble_sort_cocktail_round1.png)  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/bubble_sort_cocktail_round2.png)  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/bubble_sort_cocktail_round3.png)  



``` java
private static void playBubbleUsingCocktail(int[] arr) {
    int compareCount = 0;
    int len = arr.length;
    for (int i = 0; i < len / 2; i++){
        // 1. Left to right.
        boolean isSort = true;
        for (int j = i + 1; j <= len - i - 1; j++){
            compareCount++;
            if (arr[j] < arr[j - 1]){
                Utils.swap(arr, j, j - 1);
                isSort = false;
            }
        }
        if (isSort){
            break;
        }

        // 2. Right to left.
        isSort = true;
        for (int j = len - 1 - i - 1; j > i; j--){
            compareCount++;
            if (arr[j] < arr[j - 1]){
                Utils.swap(arr, j, j - 1);
                isSort = false;
            }
        }
        if (isSort){
            break;
        }
    }
    System.out.println(compareCount);
}
```

## Reference  
- `《漫画算法 - 小灰的算法之旅》 - 4.2 什么是冒泡排序`  
- [Sorting](http://sorting.at){:target="_blank"}