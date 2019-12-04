---
layout: post
title: Shell - 更多bash shell命令
category: shell
tags: [shell]
excerpt: More bash shell commands
---

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/more_bash_shell_commands.png)

## 如何查看磁盘容量？  

- 如何查看磁盘容量概况？

`df -h`  

其中`df`代表`Disk Free`，`h`参数代表`human-readable`，系统帮你做了进位，从而更具备可读性。  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/disk_free.png)  

- 如何查看当前目录下，最大的`10`个文件信息？  

``` shell
du -sh * | sort -r | head -n 10
// -s, --summarize: 也就是说，同一个目录只算一个元素。
// -h, --human-readable
// -r, --reverse
// -n, --lines = [-]num
```

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/du_top_10.png)  

## 处理数据文件  

- 如何搜索文件中的数据？  

``` shell
grep [option] pattern file
// -n, --lines = [-]num
// -c, --count
```

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/grep.png)  

- 如何压缩/解压文件？  

`gzip/gunzip file_name`

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/gzip.png)  

## `Reference`  
- 《Linux命令行与Shell脚本编程大全·第三版》 - 第四章(更多的bash shell命令)   