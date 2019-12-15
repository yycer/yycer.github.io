---
layout: post
title: Shell - 结构化命令
category: shell
tags: [shell]
excerpt: Shell Structure Commands
---


## `If-then-elif-then-else`  

先来看下简单的例子：  


``` shell
num=$1
if [ $num -eq 36 ]
then
    echo "Name is yyc."
elif [ $num -eq 10 ]
then
    echo "Name is frankie."
else
    echo "Other person."
fi

---------output---------
λ sh structure_commands.sh 36
Name is yyc.

λ sh structure_commands.sh 10
Name is frankie.

λ sh structure_commands.sh 12
Other person.
```

## `Test condition`  

- 什么是`Test condition`？  

> 上面例子中，方括号中的内容就是。主要的作用就是检查文件的类型和比较值。  

- `[]`和`[[]]`的区别是什么？  

先来看张图：  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/structure_commands_%5B%5D_vs_%5B%5B%5D%5D.png)

首先，`[[]]`仅在`Bash`、`Zsh`和`Ksh`中可用。  
同时，`[[]]`更加强大，体现在转义、模式匹配`Pattern Match`和正则匹配`Regular Expression Pattern`。  


接下来跟着上图中的六个特性，一一展开：  

- `String Comparison`  

``` shell
# Need to escape
# if [ "frankie" \< "yyc" ]
if [[ "frankie" < "yyc" ]]
then
    echo "f comes before y"
else
    echo "f comes after y"
fi

---------output---------
λ sh structure_commands.sh
f comes before y
```

- `Integer Comparison`  

``` shell
# if [ 10 -gt 5 ]
if [[ 10 -gt 5 ]]
then
    echo "10 > 5"
fi

---------output---------
λ sh structure_commands.sh
10 > 5
```

- `Condition Evaluation`  

``` shell
# if [ "frankie" \< "yyc" -a 10 -gt 5 ]
if [[ "frankie" < "yyc" && 10 -gt 5 ]]
then
    echo "f comes before y and 10 > 5"
fi

# if [ "frankie" \< "yyc" -o $1 -gt 5 ]
if [[ "frankie" < "yyc" || $1 -gt 5 ]]
then
    echo "f comes before y or ${1} > 5"
fi

---------output---------
λ sh structure_commands.sh 1
f comes before y and 10 > 5
f comes before y or 1 > 5
```


- `Expression Grouping`  


``` shell
λ ls
1.sh                     dirTest/        set.sh
2.sh                     error.txt       structure_commands.sh
arithmetic_operation.sh  fileTest        test.sh

# if [ -f fileTest -a \( -d dirTest -o -d dirTest1 \) ]
if [[ -f fileTest && ( -d dirTest || -d dirTest1 ) ]]
then
    echo "fileTest is a file and dirTest is a directory."
fi

---------output---------
λ sh structure_commands.sh
fileTest is a file and dirTest is a directory.
```

注意喔，下面两个特性`Pattern、RegExp Match`是`[[]]`专有的。  

- `Pattern Matching`  

``` shell
if [[ $1 = y* ]]
then
    echo "${1} starts with y."
else
    echo "${1} does not start with y."
fi

---------output---------
λ sh structure_commands.sh yyc
yyc starts with y.

λ sh structure_commands.sh frankie
frankie does not start with y.
```

- `Regular Expression Matching`  


``` shell
if [[ $1 =~ y[0-9] ]]
then
    echo "${1} starts with y and follows by a number."
else
    echo "${1} does not start with y or follow by a number."
fi

---------output---------
λ sh structure_commands.sh y123
y123 starts with y and follows by a number.

λ sh structure_commands.sh f123
f123 does not start with y or follow by a number.
```

## `Case`  

``` shell
case $1 in
    "yyc" | "frankie" ) echo "l am a computer programmer.";;
    "Alice" ) echo "$1 is a doctor.";;
    "Mike"  ) echo "$1 is a driver.";;
    *       ) echo "$1 has another job."
esac

---------output---------
λ sh structure_commands.sh yyc
l am a computer programmer.

λ sh structure_commands.sh Alice
Alice is a doctor.

λ sh structure_commands.sh Mike
Mike is a driver.

λ sh structure_commands.sh Jack
Jack has another job.
```

## `For`  

- 常规用法  

``` shell
name_arr=("yyc", "asan", "pangzi")
for name in ${name_arr[*]}
do
    echo "Name is $name"
done

---------output---------
Name is yyc,
Name is asan,
Name is pangzi
```

- C语言风格嵌套循环  

``` shell
for (( i = 0; i < 3; i++ ))
do
    echo "i = $i"
    for (( j = 0; j < 3; j++ ))
    do 
        echo "  j = $j"
    done
done

---------output---------
λ sh structure_commands.sh
i = 0
        j = 0
        j = 1
        j = 2
i = 1
        j = 0
        j = 1
        j = 2
i = 2
        j = 0
        j = 1
        j = 2
```


## `While`  

``` shell
num=$1
while [ $num -gt 0 ]
do
    echo "\$num = $num"
    (( num-- ))
done

---------output---------
λ sh structure_commands.sh 3
$num = 3
$num = 2
$num = 1
```

## `Reference`  
- [What is the difference between test, \[ and \[\[ ?](http://mywiki.wooledge.org/BashFAQ/031){:target="_blank"}  
- [test, \[, \[\[](https://www.mkssoftware.com/docs/man1/test.1.asp){:target="_blank"}  
- 《Linux命令行与Shell脚本编程大全·第三版》 - 第十二章(结构化命令)  
- 《Linux命令行与Shell脚本编程大全·第三版》 - 第十三章(更多的结构化命令)  
