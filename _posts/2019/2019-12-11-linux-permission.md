---
layout: post
title: Shell - 理解Linux文件权限
category: shell
tags: [shell]
excerpt: Understand Linux object permission
---

## 理解文件权限  

首先，我们来创建一个新的文件，看下它的默认权限是什么？  

``` shell
yyc16@ubuntu:~/Documents$ touch 1.txt
yyc16@ubuntu:~/Documents$ ls -l 1.txt 
-rw-rw-r-- 1 yyc16 yyc16 0 Dec 11 06:25 1.txt
```

重点看下`-rw-rw-r--`，它由四部分组成：  

- 第一个字段代表对象的类型，比如：`-`代表文件，`d`代表目录。  
- 后面三三一组，分别代表`对象的属主`、`对象的属组`、`系统其它用户对于该对象的权限`。其中`[r:readable, w:writable, x:executable]`，分别代表可读、可写、可执行。  

注意后面三组可按照八进制计算，也就是`664`。  

那么问题来了，为什么文件默认的权限是`664`？  

这个时候就需要提到`umask`命令，它控制着所创建文件和目录的默认权限。  

``` shell
yyc16@ubuntu:~/Documents$ umask
0002
```

`umask`明明代表`0002`，和`664`有什么关系？  

其实，`0002`是由两部分组成，第一位叫粘着位`(sticky bit)`，后三位代表`umask`八进制掩码值。对于文件来说，全权限是`666`(所有用户都有读、写权限)，对于目录来说是`777`(所有用户都有读、写、执行权限)。   

因此`664 = 666 - 002, 775 = 777 - 002`。  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/mode_calculate_default_mode.png)


## 其他  

- 为什么要使用`Linux`组？  

> 便于一群用户安全地进行交互。  


- 如何改变文件或目录的权限？  

``` shell
chmod [option] mode file/directory
```

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/mode_chmod755.png)

## Reference  
- 《Linux命令行与Shell脚本编程大全·第三版》 - 第七章(理解Linux文件权限) 
