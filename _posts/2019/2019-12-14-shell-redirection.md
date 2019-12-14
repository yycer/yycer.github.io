---
layout: post
title: Shell - 重定向
category: shell
tags: [shell]
excerpt: Shell redirection
---

## 如何将正常结果存入文件中？   

- 覆盖式`>`  

``` shell
echo "My name is frankie." > name.txt

---------output---------
λ sh redirection.sh
λ cat name.txt
My name is frankie.
```

- 追加式`>>`   

``` shell
echo "l am from Shanghai." >> name.txt

---------output---------
λ sh redirection.sh
λ cat name.txt
My name is frankie.
l am from Shanghai.
```

更加清晰地理解覆盖式：  

``` shell
echo "Cover all." > name.txt

---------output---------
λ sh redirection.sh
λ cat name.txt
Cover all.
```

## 如何将异常结果存入文件中？   

- 若仍使用`>`，也就是`1>` `(stand output)`  

``` shell
// The error is only displayed in the terminal.
echoo "My name is frankie." > error.txt

---------output---------
λ sh redirection.sh
redirection.sh: line 9: echoo: command not found
λ cat error.txt
```

-  将错误以覆盖式写入文件  

``` shell
// 2 represents error output.
echoo "My name is frankie." 2> error.txt

---------output---------
λ sh redirection.sh
λ cat error.txt
redirection.sh: line 12: echoo: command not found
```

-  将错误以追加式写入文件  

``` shell
echoo "l am from Shanghai." 2>> error.txt

---------output---------
λ sh redirection.sh
λ cat error.txt
redirection.sh: line 12: echoo: command not found
redirection.sh: line 13: echoo: command not found
```

## 如何将正常与异常的结果一起存入文件中？   

- 覆盖式  

``` shell
echoo "My name is frankie." &> log.txt

---------output---------
λ sh redirection.sh
λ cat log.txt
redirection.sh: line 16: echoo: command not found

echo "l am from Shanghai." &> log.txt
-------------------------
λ sh redirection.sh
λ cat log.txt
l am from Shanghai.
```

- 追加式  

``` shell
echoo "My name is frankie." >> log.txt 2>&1
echo "l am from Shanghai." >> log.txt 2>&1

---------output---------
λ sh redirection.sh
λ cat log.txt
redirection.sh: line 19: echoo: command not found
l am from Shanghai.
```


