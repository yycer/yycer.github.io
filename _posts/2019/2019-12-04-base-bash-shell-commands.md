---
layout: post
title: Shell - 基本的bash shell命令
category: shell
tags: [shell]
excerpt: Basic bash shell commands
---

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/Base_bash_shell_commands.png)

## 显示文件和目录列表  

有一点需要注意：`Linux用正斜杠划分目录，而不是反斜杠`。  

- 如何判断是文件还是目录？  

第一种方式是通过`ls -l`，显示的第一个字符若是`-`，则代表文件，`d`代表目录`(Directory)`。  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/ls_l.png)

第二种方式是通过`F(Classify)`参数： `ls -F`，若末尾带正斜杠，则代表目录，什么都没，就代表是文件。  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/classify.png)  


- 如何过滤输出列表？  

`ls -l filter_condition`

- 如何显示指定目录下的文件或子目录？  

`ls -l specified_path`  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/specify_path.png)  


## 处理文件  

- 如何创建一个空文件？  

`touch file_name`  

- 如何复制文件？  

`cp source destination`  

- 如何移动或修改文件名？  

``` shell
mv file_name new_file_name
mv file_name destination
```

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/mv.png)

- 如何删除文件？  

``` shell
rm -i file_name(Interactive)
rm -f file_name(Carefully!)
```


## 处理目录  


- 如何创建目录？  

`mkdir directory_name`  


- 如何删除目录？  

`rmdir directory_name`  


## 查看文件内容  

- 查看文件的三种方式？  

```shell
cat
more
less
```

- 如何查看部分文件内容？  

```shell
head -num file_name(Default 10 lines)
tail -num file_name(Default 10 lines)
```

## `Reference`  
- 《Linux命令行与Shell脚本编程大全·第三版》 - 第三章(基本的bash shell命令)   

