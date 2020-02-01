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


`Reference`  

- [Recursively counting files in a Linux directory](https://stackoverflow.com/questions/9157138/recursively-counting-files-in-a-linux-directory){:target="_blank"}