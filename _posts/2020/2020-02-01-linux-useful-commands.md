---
layout: post
title: Linux - 实用命令
category: shell
tags: [shell]
excerpt: Linux Useful Commands
---

## `find | wc`  

如何递归地统计一个目录下的所有文件？  

``` shell
find DIR_NAME -type f | wc -l
```


## `CTRL + D`  

`Ubuntu`中关闭一个终端的快捷键是什么？  


## `Grep`  

举个例子，我之前写过一篇关于`ArrayDeque`容器的文章，但是我忘记是哪篇了。  

于是，我在整个博客文件夹中进行如下搜索：  

`grep -rnw . -e 'ArrayDeque'`  

- `.`: stands for the current directory.`[当前目录]`    
- `r`: is recursive.`[递归查找]`  
- `n`: is line number.`[行号]`    
- `w`: stands for match the whole word.`[完全匹配]`      

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/linux_grep_arraydeque.png)  



`Reference`  

- [Recursively counting files in a Linux directory](https://stackoverflow.com/questions/9157138/recursively-counting-files-in-a-linux-directory){:target="_blank"}
- [What is keyboard shortcut to close terminal window? [duplicate]](https://askubuntu.com/questions/586026/what-is-keyboard-shortcut-to-close-terminal-window/586039){:target="_blank"}