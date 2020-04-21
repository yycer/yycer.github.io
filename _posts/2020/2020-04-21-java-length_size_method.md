---
layout: post
title: Java - 区分length属性、length()和size()方法
category: tool
tags: [tool]
excerpt: lenght property, length() and size()
---

最近在`leetcode`刷题的时候，老是混淆`length`属性、`length()`和`size()`方法，因此写篇文章记录一下。  


## length - 数组  

首先，`length`属性用在哪里？  

数组！  

举个例子：  

``` java
int[] arr = {1, 2, 3};
System.out.println("The length of arr is " + arr.length);
```

## length() - 字符串  

再来，`length()`这个方法又是谁用的？  

字符串！  

``` java
String str = "abc";
System.out.println("The length of str is " + str.length());
```

## size() - 容器  

最后，`size()`这个方法谁用的？  

容器！  


``` java
List<Integer> list = new ArrayList<>();
list.add(1);
list.add(2);
System.out.println("The size of list is " + list.size());
```

## Reference  
- [Java 中的 length 、length()、size()一次摸透透](https://medium.com/cubemail88/java-%E4%B8%AD%E7%9A%84-length-length-size-%E4%B8%80%E6%AC%A1%E6%91%B8%E9%80%8F%E9%80%8F-24b82cb41e22){:target="_blank"}  
- [Java length() 方法，length 属性和 size() 方法的区别](https://www.runoob.com/note/20596){:target="_blank"}  