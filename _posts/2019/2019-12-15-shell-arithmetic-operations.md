---
layout: post
title: Shell - 算数运算的四种方式
category: shell
tags: [shell]
excerpt: Shell Arithemtic Operations
---

## 单词区分  

``` shell
1. (双)圆括号: (Double) Parentheses.
2. (双)方括号: (Double) Square Brackets.
3. 反引号: Backticks.
4. 反斜杠: Backslash.
```

## `let`  

先来看下定义：  

> Carry out arithmetic operations on variables.  

也就是说，它会将算数运算的结果赋值给变量。  


``` shell
let a=12
echo "a = $a"
# Double quotes and spaces make it more readable.
let "b = 12"
echo "b = $b"

---------output---------
λ sh arithmetic_operation.sh
a = 12
b = 12
```

再来看几个简单的算数运算：  

``` shell
let "a = 12"
echo "a = $a"

let "a /= 4"
echo "a / 4 = $a"

let "a *= 8"
echo "a * 8 = $a"

let "a %= 5"
echo "a % 5 = $a"

let a++
echo "a++ = $a"

---------output---------
λ sh arithmetic_operation.sh
a = 12
a / 4 = 3
a * 8 = 24
a % 5 = 4
a++ = 5
```

还有一点：`let`命令的`exit status`很有意思：  

``` shell
let "b = 0"
echo "b = 0, \$? = $?"

// When executing let command, b = 0
let b++
echo "b++, \$? = $?"

// When executing let command, b = 1
let b++
echo "b++, \$? = $?"

---------output---------
λ sh arithmetic_operation.sh
b = 0, $? = 0
b++, $? = 1
b++, $? = 0
```

原因如下：  

> If the last ARG evaluates to 0, let returns 1; let returns 0 otherwise.  

证明下：  

``` shell
let "c = 0"
echo "let c = 0, \$? = $?"

---------output---------
λ sh arithmetic_operation.sh
let c = 0, $? = 1
```



## `expr`  

先来看下定义：  

> All-purpose expression evaluator: Concatenates and evaluates the arguments according to the operation given(arguments must be separated by spaces).  
> Operations may be arithmetic, string, or logical.

下面从`arithmetic`, `string`, `logical`这三个点详细说明下：  

- `Arithmetic operator`  

``` shell
a=`expr 3 + 5`
echo "a = 3 + 5 = $a"

a=`expr $a + 1`
echo "\$a + 1 = $a"

// Must have a backslash before *.
b=`expr 3 \* 5`
echo "3 * 5 = $b"

---------output---------
λ sh arithmetic_operation.sh
a = 3 + 5 = 8
$a + 1 = 9
3 * 5 = 15
```

- `Logical operator`  

`expr`中逻辑运算的结果也很有意思：  

> Return 1 if true, 0 if false, is opposite of normal bash convention.  

也就是说，如果是`true`，它返回`1`，否则返回`0`，和常规的`exit status`相反。  

``` shell
let "b = 30"
c=`expr $b \> 10`
echo "$b > 10, exit status = $c"

let "x = 1"
let "y = 2"
a=`expr $x = $y`
echo "$x = $y, exit status = $a"

---------output---------
0 > 10, exit status = 1
1 = 2, exit status = 0
```

- `String operator`  

其实我觉得，只有当你需要处理字符串时，才应该用`expr`，因为算数运算完全可以用`let`或`Arithmetic Expansion`、逻辑处理可以用`Test Condition`。  


``` shell
a="123yyc123"
echo "a = $a"

a_length=`expr length $a`
echo "The length of a is $a_length"

# Index begin with 1.
a_23_index=`expr index $a 23`
echo "The position of first 23 in substring is $a_23_index"

a_substr_1_3=`expr substr $a 1 3`
echo "Substring of $a, starting at position 1, and 3 chars long is $a_substr_1_3"

# The default behavior of the match operations is to search for the specified match at the beginning of the string.
# The : operator can substitute for match
# b=`expr match $a '[0-9]*'`
b=`expr $a : '[0-9]*'`
echo "The number of digits at the beginning of $a is $b"

# c=`expr match $a '\([0-9]*\)'`
c=`expr $a : '\([0-9]*\)'`
echo "The digits at the beginning of $a are $c"

---------output---------
λ sh arithmetic_operation.sh
a = 123yyc123
The length of a is 9
The position of first 23 in substring is 2
Substring of 123yyc123, starting at position 1, and 3 chars long is 123
The number of digits at the beginning of 123yyc123 is 3
The digits at the beginning of 123yyc123 are 123
```


## `Double-parentheses Construct: Arithemtic Expansion(())`  

``` shell
# Within double parentheses, parameter dereferencing is optional
let "a = 1"
echo "a = $a"

a=$(( $a + 1 ))
echo "a = \$a + 1 = $a"

a=$(( a + 1 ))
echo "a = a + 1 = $a"

# You may alse use operations within double parentheses without assignment.
(( a++ ))
echo "a = a++ = $a"

x=2
y=5
# You do not need backslash before *.
b=$(( x * y ))
echo "$x * $y = $b"

---------output---------
λ sh arithmetic_operation.sh
a = 1
a = $a + 1 = 2
a = a + 1 = 3
a = a++ = 4
2 * 5 = 10
```

## `Integer Expansion - $[]`  

> Note that this usage is deprecated, and has been replaced by the Double-Parentheses construct.  


## `Summary`  

最后总结下：  

- 算数运算，推荐使用`Double Parentheses`，可读性更强，同时避免转义问题。  
- 逻辑运算，推荐使用`Test Condition`，也就是`[]`或`[[]]`，两者的区别会单独写篇文章讲述。  
- 字符串处理，推荐使用`expr`，如`length`、`substr`、`index`等。
- `$[]`，已废弃，不建议使用。  


## `Reference`  
- [let](http://tldp.org/LDP/abs/html/internal.html#LETREF){:target="_blank"}
- [expr
](http://tldp.org/LDP/abs/html/moreadv.html#EXPRREF){:target="_blank"}
- [Chapter 13. Arithmetic Expansion](http://tldp.org/LDP/abs/html/arithexp.html#ARITHEXPREF){:target="_blank"}