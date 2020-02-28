---
layout: post
title: Java - 枚举 & switch语句
category: java-basic
tags: [spring, java]
excerpt: Java enum and switch statements.
---

最近有种将几个知识点连成线的感觉。  

本文来介绍下`枚举类`和`switch`控制语句。  


## `enum`  

为什么要用枚举类？  

换个问题，不用枚举类，你会如何表示四季？  

``` java
public static final int SEASON_SPRING = 1;
public static final int SEASON_SUMMER = 2;
public static final int SEASON_FALL   = 3;
public static final int SEASON_WINTER = 4;
```

这可能会导致一些问题，如：不小心将上面的变量做计算操作、含义表达的不明确等。  

我们用枚举类表示下：  

``` java
public enum SeasonEnum {
    SPRING, SUMMER, FALL, WINTER;
}
```

注意一个硬性规定： `所有实例必须在枚举类开始显示列出！`  


第二个问题，枚举既然也是类，那么能否使用成员变量？若能，如何使用？  

答案肯定是可以的，我们举个实际的例子：  

``` java
@Getter
public enum OrderErrorEnum {

    ORDER_AMOUNT_ERROR("RCOE0001", "订单金额不能小于零"),
    ORDER_TYPE_ERROR  ("RCOE0002", "该用户不支持该订单类型"),
    ;

    private final String errorCode;
    private final String errorMsg;

    OrderErrorEnum(String errorCode, String errorMsg){
        this.errorCode = errorCode;
        this.errorMsg = errorMsg;
    }
}
```

- 枚举类中的实例之间以逗号分隔，最后以分号结尾。  
- 实例都是以一种类似构造函数的方式构建的`实例名(参数1, 参数2)`  
- 实例一定出现在枚举类的开头！  


## `switch`  

同样的问题，为什么要用`switch`？  

我相信相同的功能，你一定能用`if...else...`实现，所以我的理解是：`switch`的写法层次上更加清楚。  

先来看下`switch语句`的语法格式：  

``` java
switch (expression){
    case condition1:
        statements
        break;
    case condition2:
        statements
        break;
    ...
    default:
        statements;
}
```

来说下`expression`它仅支持六种类型：`char`、`byte`、`short`、`int`、`enum`、`String[Java7]`。  

结合枚举类举个例子：  


``` java
@Test
void simpleEnumTest(){
    Utils.describeSeason(SeasonEnum.SPRING);
}

public static void describeSeason(SeasonEnum s){
        switch (s){
            case SPRING:
                System.out.println("Spring coming.");
                break;
            case SUMMER:
                System.out.println("Summer coming.");
                break;
            case FALL:
                System.out.println("Fall coming.");
                break;
            case WINTER:
                System.out.println("Winter coming.");
                break;
            default:
                throw new RuntimeException("Please enter valid season!");
        }
    }
```



## `Reference`  
- `《疯狂Java讲义》 - 4.2.2 Java11改进的switch分支语句`  
- `《疯狂Java讲义》 - 6.9 枚举类`  