---
layout: post
title: 常见排序算法总结
category: algorithm
tags: [algorithm]
excerpt: 冒泡、选择、插入、希尔、归并、快速排序
---
## Bubble Sort(冒泡排序)  
- 核心: 小的数字依次从最后往前冒。
- 优点: 稳定。
- 缺点: 比较和交换次数过多。

```java
 public void bubbleSort(int[] a){
    if (a.length < 1){
        System.out.println("Please enter valid array!");
    }

    for (int i = 0; i < a.length - 1; i++){
        for (int j = a.length - 1; j > i; j--){
            if (less(a[j], a[j - 1])){
                swap(a, j, j - 1);
            }
        }
    }
}
```

<br>
<br>
## Selection Sort(选择排序)  
- 核心: 选择剩下元素中最小的。
- 优点: 相比于冒泡排序，每轮中的交换次数有所减少。
- 缺点: 比较次数还是太多，不稳定如`{5, 8, 5, 2, 9}`。

```java
public void selectionSort(int[] a){
    if (a.length < 1){
        System.out.println("Please enter valid array!");
    }

    for (int i = 0; i < a.length - 1; i++){
        int minIndex = i;
        for (int j = i + 1; j < a.length; j++){
            if (less(a[j], a[minIndex])){
                minIndex = j;
            }
        }
        if (minIndex != i){
            swap(a, minIndex, i);
        }
    }
}
```

<br>
<br>
## Insertion Sort(插入排序)  

- 核心: 抓牌，小的牌插到前面去。
- 优点: 稳定。
- 缺点: 交换与比较次数仍过多。

```java
public void insertionSort(int[] a){
    if (a.length < 1){
        System.out.println("Please enter valid array!");
    }

    for (int i = 1; i < a.length; i++){
        int j = i;
        while (j > 0){
            if (less(a[j], a[j - 1])){
                swap(a, j, j - 1);
            }
            j--;
        }
    }
}
```
<br>
<br>
## Shell Sort(希尔排序)  
- 核心: `gap`。
- 优点: 插入排序的升级版。
- 缺点: 不稳定。

```java
public void shellSort(int[] a){
    if (a.length < 1){
        System.out.println("Please enter valid array!");
    }

    int gap = a.length >> 1;
    while (gap >= 1){
        for (int i = gap; i < a.length; i++){
            int j = i;
            while (j >= gap){
                if (less(a[j], a[j - gap])){
                    swap(a, j, j - gap);
                }
                j -= gap;
            }
        }
        gap >>= 1;
    }
}
```

<br>
<br>
## Merge Sort(归并排序)  
- 核心: 分治思想。
- 优点: 稳定。

```java
public void mergeSort(int[] a){
    if (a.length < 1){
        System.out.println("Please enter valid array!");
    }

    sep(a, 0, a.length - 1);
}

/**
 * 分
 */
public void sep(int[] a, int l, int r){
    if (l >= r){
        return;
    }

    int mid = (l + r) >> 1;
    sep(a, l, mid);
    sep(a, mid + 1, r);
    merge(a, l, mid + 1, r);
}

/**
 * 治
 */
public void merge(int[] a, int l, int mid, int r){
    int[] leftArray  = new int[mid - l];
    int[] rightArray = new int[r - mid + 1];

    for (int i = l; i < mid; i++){
        leftArray[i - l]    = a[i];
    }
    for (int j = mid; j <= r; j++){
        rightArray[j - mid] = a[j];
    }

    int i = 0, j = 0, k = l;
    while (i < leftArray.length && j < rightArray.length){
        if (less(leftArray[i], rightArray[j])){
            a[k++] = leftArray[i++];
        } else {
            a[k++] = rightArray[j++];
        }
    }
    while (i < leftArray.length){
        a[k++] = leftArray[i++];
    }

    while (j < rightArray.length){
        a[k++] = rightArray[j++];
    }
}
```

<br>
<br>
## Quick Sort(快速排序)  

- 核心: `pivot`基点。
- 缺点: 不稳定。

```java
 public void quickSort(int[] a, int l, int r){
    if (a.length < 1 || l >= r){
        return;
    }

    int lt = l, gt = r, i = l + 1;
    int pivot = l;

    while (i <= gt){
        if (less(a[i], a[pivot])){
            swap(a, i++, lt++);
        } else if (greater(a[i], a[pivot])){
            swap(a, gt--, i);
        } else {
            i++;
        }
    }

    quickSort(a, l, lt - 1);
    quickSort(a, gt + 1, r);
}
```

<br>
<br>
## 基础方法
```java
 public void swap(int[] a, int i, int j){
    int tmp = a[i];
    a[i]    = a[j];
    a[j]    = tmp;
}

public boolean less(int a, int b){
    return a < b;
}

public boolean greater(int a, int b){
    return a > b;
}
```
