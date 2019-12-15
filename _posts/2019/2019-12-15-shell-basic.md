---
layout: post
title: Shell - Shell编程基础
category: shell
tags: [shell]
excerpt: Shell Basic
---

## 如何知道当前`Shell`版本？  

``` shell
echo $0
echo $SHELL

---------output---------
/usr/bin/bash
```
## 如何打印数据？  

``` shell
echo "yyc"

---------output---------
yyc
```

- 如何你想显示单或双引号，怎么办？  

>可以用另一种将其括起来。    


``` shell
echo "My name's yyc."
echo 'He is a "Super hero".'

---------output---------
My name's yyc.
He is a "Super hero".
```

## 用户变量  

有如下几点规则：  

- 变量名、等号、变量值之间不能有空格。  
- 变量名只能以字母或下划线开头。  
- 变量名区分大小写。  
- 生命周期为整个脚本。  
- 当引用一个变量时，需要用美元符号。  
- 对一个变量赋值时，不需要美元符号。  

``` shell
user_name="yyc"
echo "user_name = $user_name"

echo "User_name = $User_name"

user_name="frankie"
echo "user_name = $user_name"

---------output---------
λ sh base_shell.sh
user_name = yyc
User_name =
user_name = frankie
```

## 命令替换  

- 什么是命令替换？  

> In computing, command substitutionn is a facility and allows a command to be run and its output to be pasted back on the command line as arguments to another command.  

也就是说，一个命令的返回值可以作为另一个命令的输入。  

- 命令替换的实现方式有哪几种？    

主要有两种：  

- 反引号  
- `$()`  

推荐使用后者，因为`$()`可避免转义，同时嵌套的方式更为直观。  


``` shell
# 3.1 backslash
echo `echo \\a`
echo $(echo \\a)

# 3.2 nesting command substitutions
echo "This year is $(date +%Y)"
# Command substitution and Arithmetic operations.
echo "Next year is $(( $(date +%Y) + 1 ))"

echo "This year is `date +%Y`"
echo "Next year is `expr \`date +%Y\` + 1`"

---------output---------
a
\a
This year is 2019
Next year is 2020
This year is 2019
Next year is 2020
```


## 管道  

作用是组合更多命令。  

- 以倒序的方式输出将当前目录下最大的三个文件。  

``` shell
# -s, --size
# -h, --human-readable
# -n, --numeric-sort
# -r, --reverse
# -n, --lines=[-]Num
ls -sh * | sort -n -r | head -n 3

---------output---------
174M 深入理解Spring+Cloud与微服务构建.pdf
48M Spring实战（第4版）.pdf
46M Spring Cloud微服务实战.pdf
```

## 如何进行浮点运算？  

``` shell
# bc: binary calculator.
echo "scale = 3; 5 / 3" | bc

---------output---------
1.666
```

## 退出脚本  

`Shell`中运行的每个命令都会返回一个退出状态码`Exit Status`。  

- 如何查看上个命令产生的退出状态码？  

> $?  

- 如何判断上个命令是否执行成功？  

> 若返回0，则代表执行成功。    



- `exit`命令的作用是什么？  

> 指定退出状态码，范围为`0 ~ 255`。  



## `Reference`  
- 《Linux命令行与Shell脚本编程大全·第三版》 - 第十一章(Shell脚本编程基础)  


