---
layout: post
title: Shell - 函数
category: shell
tags: [shell]
excerpt: Function
---


## 创建函数的两种方式？  

``` shell
// You must add a space before the curly braces.
function getName {
    echo "yyc"
}

getAge() {
    echo 23
}
```

## 如何调用函数？  

``` shell
// No parentheses required.
getName
getAge

---------output---------
+ getName
+ echo yyc
yyc
+ getAge
+ echo 23
23
```

## 如何将函数的返回值保存到一个变量中？  

``` shell
// command substitution.
age=$(getAge)
echo "My age is $age"

---------output---------
++ getAge
++ echo 23
+ age=23
+ echo 'My age is 23'
My age is 23
```

## 如何往函数中传入参数？  

``` shell
function getOrderId {
    echo "The orderId is $1"
}

getOrderId "3b378a65-c176-4e9e-af38-c6d638a2564e"

---------output---------
+ getOrderId 3b378a65-c176-4e9e-af38-c6d638a2564e
+ echo 'The orderId is 3b378a65-c176-4e9e-af38-c6d638a2564e'
The orderId is 3b378a65-c176-4e9e-af38-c6d638a2564e
```

## 如何在函数中创建局部变量？  

``` shell
function localVariableTest {
    local var1=10
    echo "var1 in function is $var1"
}
localVariableTest

---------output---------
+ localVariableTest
+ local var1=10
+ echo 'var1 in function is 10'
var1 in function is 10
```

## 创建库  

首先，为什么要创建库？  

> 我的理解是，为了使多个脚本共用一种状态或行为。    

那么如何实现呢？  

为了方便起见，在当前目录添加`commonFunctions.sh`脚本，内容如下：  

``` shell
function add {
    echo $[ $1 + $2 ]
}
```

然后在另一个脚本中执行如下命令：  

``` shell
. ./commonFunctions.sh

var1=10
var2=20
echo $(add $var1 $var2)

---------output---------
+ . ./commonFunctions.sh
+ var1=10               
+ var2=20               
++ add 10 20            
++ echo 30              
+ echo 30               
30
```

## `Reference` 
- 《Linux命令行与Shell脚本编程大全·第三版》 - 第十七章(创建函数) 