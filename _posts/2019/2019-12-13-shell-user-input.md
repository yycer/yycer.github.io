---
layout: post
title: Shell - 用户输入
category: shell
tags: [shell]
excerpt: Shell user input
---

## 什么是位置参数`Positional parameter`？  

先来看段定义：  

> A Positional parameter is an argument specified on the command line,  
> used to launch the current process in a shell.  

也就是说，位置参数是命令行中指定传入的参数，用来保证当前脚本的正常运行。  

``` shell
#!/bin/bash
set -euxo pipefail

// represents script name.
echo $0

if [ -n $1 ]
then
    echo "\$1 = $1"
fi

---------output---------
λ sh user_input.sh yyc
+ echo user_input.sh
user_input.sh
+ '[' -n yyc ']'
+ echo '$1 = yyc'
$1 = yyc
```

## 移动变量  

综合使用`while`、`case`、`shift`。  

先来看下`shift`的定义：  

> shift [n]: shift positional parameters. Rename the positional parameters $N+1, $N+2 ... to $1, $2...  
> If N is not given, it is assumed to be 1.  

``` shell
while [ -n $1 ]
do
    case $1 in 
        -h ) echo "Found --human-readable option.";;
        -r ) echo "Found --reverse option.";;
        *  ) echo "Other options"
    esac
    shift
done

---------output---------
λ sh user_input.sh -h -r
+ '[' -n -h ']'
+ case $1 in
+ echo 'Found --human-readable option.'
Found --human-readable option.
+ shift
+ '[' -n -r ']'
+ case $1 in
+ echo 'Found --reverse option.'
Found --reverse option.
+ shift
user_input.sh: line 15: $1: unbound variable
```

## 两种特殊参数变量  

``` shell
// Count input parameters.
echo "\$# = $#"

// Print input parameters.
echo "\$* = $*"

---------output---------
λ sh user_input.sh -h -r
+ echo '$# = 2'
$# = 2
+ echo '$* = -h -r'
$* = -h -r
```

## 以`--`分离选项与参数  

``` shell
while [ -n $1 ]
do
    case $1 in
        -a ) echo "Found -a option.";;
        -h ) echo "Found -h option.";;
        -s ) echo "Found -s option.";;
        # bingo
        -- ) shift;break;;
        *  ) echo "Found others optianl";;
    esac
    shift
done
        
for i in $@
do
    echo "The parameter is $i"
done

---------output---------
λ sh user_input.sh -h -r -- yyc frankie
+ '[' -n -h ']'
+ case $1 in
+ echo 'Found -h option.'
Found -h option.
+ shift
+ '[' -n -r ']'
+ case $1 in
+ echo 'Found others optianl'
Found others optianl
+ shift
+ '[' -n -- ']'
+ case $1 in
+ shift
+ break
+ for i in $@
+ echo 'The parameter is yyc'
The parameter is yyc
+ for i in $@
+ echo 'The parameter is frankie'
The parameter is frankie
```

## 获得用户输入  

- 用户输入
``` shell
// -p represents prompt.
read -p "Please enter the name: " name
echo "$name"

---------output---------
+ read -p 'Please enter the name: ' name
Please enter the name: yyc
+ echo yyc
yyc
```

- 如何设置输入超时？  

``` shell
// -t: represents time out and return failure if a complete line of input is not read within TIMEOUT seconds.
if read -t 5 -p "Please enter the name: " name
then
    echo "The name is $name"
else
    echo "Too slowly."
fi

---------output---------
λ sh user_input.sh
+ read -t 5 -p 'Please enter the name: ' name
Please enter the name: + echo 'Too slowly.'
Too slowly.
```

- 如何隐式输入？比如：输入密码。  

``` shell
// -s: represents seal, do not echo input coming from a terminal.
read -s -p "Please enter the passsword: " passsword
echo "$passsword"

---------output---------
λ sh user_input.sh
+ read -s -p 'Please enter the passsword: ' passsword
Please enter the passsword: + echo 123456
123456
```

## `Reference`  
- 《Linux命令行与Shell脚本编程大全·第三版》 - 第十四章(处理用户输入) 