---
layout: post
title: Java - [面向对象]方法和变量
category: java-basic
tags: [spring, java]
excerpt: Java OOP Method And Variable
---

本文梳理下方法，主要涉及如下几个方面：  

- 参数传递机制  
- 可变形参  
- 递归方法  


## 参数传递机制  

`Java`中方法的参数参数方式只有一种： `值传递`  

那么什么是`值传递`？  

`值传递：将实际参数值的副本(复制品)传入方法内，而参数本身不受任何影响`。  

反正我第一次看这种文绉绉的定义，都是云里雾里的，下面我们结合例子来理解下。  

首先，我们要确定一点：传入参数的类型一共有两种，要么`基础类型`，要么`引用类型`。  

好的，我们依次说明下。  

先说`基础类型`：  

``` java
public static void swap(int a, int b){
    int tmp = a;
    a = b;
    b = tmp;
    System.out.println(String.format("in    swap(): a = %d, b = %d", a, b));
}

@Test
void primitiveTransferTest(){
    int a = 5, b = 8;
    Utils.swap(a, b);
    System.out.println(String.format("after swap(): a = %d, b = %d", a, b));
}
```

猜下`swap()`和`primitiveTransferTest()`方法中的`a`和`b`分别是多少？  


``` java
// in    swap(): a = 8, b = 5
// after swap(): a = 5, b = 8
```

再来看下前面那句话：  

`值传递：将实际参数值的副本(复制品)传入方法内，而参数本身不受任何影响`。  

也就是说，`swap()`方法中的`a`和`b`是`primitiveTransferTest`方法中的副本，无论`swap()`如何修改`a`和`b`，`primitiveTransferTest()`中的`a`和`b`是不受影响的。  

再来看张图，深入理解下：  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/method_primitive_value_transfer.png)  


再来说下`引用类型`：  

``` java
@Setter
@Getter
public class DataWrap {

    private int a;
    private int b;

    public DataWrap(int a, int b){
        this.a = a;
        this.b = b;
    }
}

public static void swapDataWrap(DataWrap dw){
    int tmp = dw.getA();
    dw.setA(dw.getB());
    dw.setB(tmp);
    System.out.println(String.format("in    swap(): a = %d, b = %d", dw.getA(), dw.getB()));
}

@Test
void referenceTransferTest(){
    DataWrap dw = new DataWrap(5, 8);
    Utils.swapDataWrap(dw);
    System.out.println(String.format("after swap(): a = %d, b = %d", dw.getA(), dw.getB()));
}
```

同样的问题：`swapDataWrap()`和`referenceTransferTest()`两个方法中`dw.a`和`dw.b`分别是什么？  


``` java
// in    swap(): a = 8, b = 5
// after swap(): a = 8, b = 5
```

`swapDataWrap()`收到`referenceTransferTest()`方法中`dw`引用的复制品，它也指向堆内存中的`DataWrap`对象。  

如下图所示：  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/method_reference_transfer_2.png)  

所以，在`swapDataWrap()`中修改`a`和`b`后，`本体`也随之改变。  


## 可变形参  

之前在翻源码的时候，经常在方法的形参列表中能看到这种写法`Type...`。  

原来这就是所谓的`可变形参`，也就是说方法允许传入数量不确定的形参。  

但是，还有有几个限制的：

- 可变形参只能放在参数列表末尾。  
- 一个方法中最多只能有一个数量可变的形参。  
- 可变形参中的类型都是一样的。  

举个例子：  

``` java
public static void paramDynamic(int a, String... names){
    for (var name: names)
        System.out.println(name);
}

@Test
void parameterDynamicTest(){
    Utils.paramDynamic(10, "frankie", "asan", "paangzi");
    // String[] names = {"frankie", "asan", "paangzi"};
    // Utils.paramDynamic(20, names);
}

// frankie
// asan
// paangzi
```

可变参数的本质就是一个`数组类型的形参`，所以你也可以传入一个数组。  


## 递归方法  

递归就是在方法内部调用自身。  

这大多数人都知道，讲个有意思的地方：  

`要往已知的方向递归。`  

来看两个例子：  

`f(0) = 1, f(1) = 4, f(n + 2) = 2 * f(n + 1) + f(n), 求f(10)。`


``` java
public static int func(int n){
    if (n < 0) throw new RuntimeException("Please enter valid number!");
    else if (n == 0) return 1;
    else if (n == 1) return 4;
    // f(n) = 2f(n - 1) + f(n - 2)
    else return 2 * func(n -1 ) + func(n - 2);
}

@Test
void recursionTest(){
    int ret = Utils.func(10);
    Assert.assertEquals(ret, 10497);
}
```


`f(20) = 1, f(21) = 4, f(n + 2) = 2 * f(n + 1) + f(n), 求f(10)。`


``` java
public static int fn(int n){
    if (n == 20) return 1;
    else if (n == 21) return 4;
    // f(n) = f(n + 2) - 2f(n + 1)
    else return fn(n + 2) - 2 * fn(n + 1);
}

@Test
void recursionTest(){
    int ret = Utils.fn(10);
    System.out.println(ret);
}
```


## 变量  

再来介绍下变量。  

`Java`中一共有几种类型的变量？  

``` java
public class FiveVariable {

    private       String instanceVariable = "instanceVariable";
    public static String staticVariable   = "staticVariable";

    public void getName(String formalParameter){
        String methodLocalVariable = "methodLocalVariable";
    }

    {
        String blockLocalVariable = "blockLocalVariable";
    }
}
```

一共有五种，分为两类：

- 成员变量  
    + 实例成员变量： `classInstanceVariable`  
    + 类成员变量： `classStaticVariable`  
- 局部变量
    + 形参： `formalParameter`  
    + 方法局部变量： `methodLocalVariable`  
    + 代码块局部变量： `blockLocalVariable`  



## `reference`  
- `《疯狂Java讲义》 - 5.2 方法详解`  