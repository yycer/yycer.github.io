---
layout: post
title: Shell - 使用Linux环境变量
category: shell
tags: [shell]
excerpt: Use Linux environment variables
---

## 如何获得环境变量？  

一共有如下三种方式：  

``` shell
1. env
2. printenv
3. set
```

我们先来说下`env`与`set`的区别。

> env: lt only sees environment variables.  
> set: Since set is a built-in command, it can also shell-local variables(including shell functions).  

也就是说，`env`的作用就是显示环境变量，而`set`的作用更广，还包含一些本地`shell`变量和函数。  

再来看下`printenv`：  

> printenv: Print the values of the specified environment variables.  
> If no variable is specified, print name and value pairs for them all.  

也就是说，当`printenv`没有指定某个或多个`(以空格分隔)`环境变量时，它的作用就和`env`一样。  


## 三种变量梳理  

- 全局环境变量  
- 局部变量  
- 函数中的局部变量  

先来说下全局环境变量，它对于所有`shell`均可用。  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/test_environment_variable.png)  


而局部变量仅对当前`shell`生效。  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/env_local_variables.png)    



## `set`命令的四个常用参数  

- `-u: Treat unset variables as an error when substituting.`  

默认情况下，遇到不存在的变量，输出的是空行。但是，我们往往期望直接报错。  

``` shell
set -u

echo $a
echo "yyc"
```

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/set_u.png)


- `-x: Print commands and their arguments where they are executed.`

`x`参数的作用是将命令以及涉及的参数打印出来，让你更加清楚脚本执行了什么。  

``` shell
set -x
name=${1}

function sayHi {
  echo "Hi ${1}"
}
sayHi $name
```

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/set_x.png)


- `-e: Exit immediately if a command exit with a non-zero status.`

首先要说下默认的错误处理机制，一旦出现错误，会打印出来，但是仍会继续往下执行。但是遇到错误的情况，我们往往期望立即停止，这就是`e`参数的由来。  

``` shell
set -e

echoo "yyc"
echo "yyc"
```

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/set_e.png)


- `-o pipefail`

当你使用`e`参数后，大多数的错误场景均会直接退出，但是遇到管道就失灵了，因此就发明了`-o pipefail`。  

``` shell
set -eo pipefail

ehcoo "yyc" | echo "yyc"
echo "franki
```

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/set_o_pipefail.png)


总结一下，以后在`shell`脚本中推荐使用`set -euxo pipefail`。  


## `Others`  

- 命名规范

1. 环境变量名建议使用全大写和下划线。  
2. 局部变量名建议使用全小写和下划线。  

- 如何设置或删除环境变量？

``` shell
// set environment variable.
export ENV_KEY=value

// unset environment variable.
unset ENV_KEY
```

## `Reference`  
- [Bash 脚本 set 命令教程](http://www.ruanyifeng.com/blog/2017/11/bash-set.html){:target="_blank"}
- [What's the difference between “env” and “set” (on Mac OS X or Linux)?](https://stackoverflow.com/questions/5657060/whats-the-difference-between-env-and-set-on-mac-os-x-or-linux){:target="_blank"}
- 《Linux命令行与Shell脚本编程大全·第三版》 - 第六章(使用Linux环境变量)   


