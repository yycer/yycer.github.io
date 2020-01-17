---
layout: post
title: 工作经验
category: working-experience
tags: []
excerpt: Working Experience
---

## `switch`  

1. 有个`state`字段，类型为`String`，其中`[1:SUCCESS, 2:FAILED, 3:PROCESSING]`，用惯了`if...else`，于是想试下`switch`。  

``` java
@Test
public void caseTest(){
    String state  = "1";
    String result = "";

    switch (state){
        case "1":
            result = "SUCCESS";
            break;
        case "2":
            result = "FAILED";
            break;
        case "3":
            result = "PROCESSING";
            break;
        default:
            System.out.println("default");
    }
    System.out.println("Result = " + result);
}
```

## `Regular Expression`  

1. 校验身份证，仅支持`16`或`18`位的数字和字母。  

``` java
@Test
public void regExpTest(){
    // Match 16 or 18 digit identity.
    // 18
    boolean matches1 = Pattern.matches("[0-9a-zA-Z]{16}|[0-9a-zA-Z]{18}", "390113198608974610");

    // 16
    boolean matches2 = Pattern.matches("[0-9a-zA-Z]{16}|[0-9a-zA-Z]{18}", "3901131986089746");

    // 18 with alpha
    boolean matches3 = Pattern.matches("[0-9a-zA-Z]{16}|[0-9a-zA-Z]{18}", "39011319860897461a");
    boolean matches4 = Pattern.matches("[0-9a-zA-Z]{16}|[0-9a-zA-Z]{18}", "39011319860897461X");

    // 16 with alpha
    boolean matches5 = Pattern.matches("[0-9a-zA-Z]{16}|[0-9a-zA-Z]{18}", "390113198608974a");
    boolean matches6 = Pattern.matches("[0-9a-zA-Z]{16}|[0-9a-zA-Z]{18}", "390113198608974X");

    // Failed
    boolean matches11 = Pattern.matches("[0-9a-zA-Z]{16}|[0-9a-zA-Z]{18}", "39011319860897461_");
    boolean matches12 = Pattern.matches("[0-9a-zA-Z]{16}|[0-9a-zA-Z]{18}", "390113198608974_");

    // Match 11 digit phone number.
    String phone = "11111222223";
    boolean matches21 = Pattern.matches("[0-9]{11}", "11111222223");

    // Failed
    boolean matches22 = Pattern.matches("[0-9]{11}", "1111122222");
    boolean matches23 = Pattern.matches("[0-9]{11}", "111112222233");
}
```


## `如何找到并杀死进程`  

``` shell
netstat -ano | find "port"
taskkill -f -pid processId
```