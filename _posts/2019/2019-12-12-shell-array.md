---
layout: post
title: Shell - 数组
category: shell
tags: [shell]
excerpt: Array
---

## 如何创建一个数组，并为其赋值？  

``` shell
arr1=(1 2 3)
```

## 如何输出数组中的所有元素？  

``` shell
echo "\$arr1 = $arr1"
echo "\${arr1[*]} = ${arr1[*]}"

---------output---------
+ arr1=(1 2 3)
+ echo '$arr1 = 1'
$arr1 = 1
+ echo '${arr1[*]} = 1 2 3'
${arr1[*]} = 1 2 3
```


## 如何遍历一个数组？  

``` shell
for x in ${arr1[*]}
do
    echo $x
done

---------output---------
+ arr1=(1 2 3)
+ for x in ${arr1[*]}
+ echo 1
1
+ for x in ${arr1[*]}
+ echo 2
2
+ for x in ${arr1[*]}
+ echo 3
3
```


## 如何修改数组中指定索引的值？  

``` shell
arr1[0]=10
echo ${arr1[*]}

---------output---------
+ arr1=(1 2 3)
+ arr1[0]=10
+ echo 10 2 3
10 2 3
```

## 如何知道数组的大小？  

``` shell
echo "The size of arr1 is ${#arr1[*]}"

---------output---------
+ arr1=(1 2 3)
+ echo 'The size of arr1 is 3'
The size of arr1 is 3
```

## `Reference`  
- 《Linux命令行与Shell脚本编程大全·第三版》 