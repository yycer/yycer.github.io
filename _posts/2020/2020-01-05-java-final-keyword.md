---
layout: post
title: Java - [面向对象]final
category: java-basic
tags: [spring, java]
excerpt: Java OOP final
---

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/final.png)  

### `2020_0306补充理解`  

首先，`final`关键字用于修饰什么？  

一共三种：`类`、`变量`和`方法`。  

`final`修饰的类不可以有子类。  

`final`修饰的方法不可以被重写。  

下面主要说下`final`修饰的变量：  


## `final`成员变量  

成员变量是随`类初始化`或`对象初始化`而初始化的。  

- 类成员变量  

必须在`声明该类成员变量`或`静态初始化代码块`中指定初始值，并且只能`二者选其一`。  


``` java
public class FinalVariableTest {
    // Method1: Specifying initial value when declaring variables. 
    private static final int count = 10;

    // Method2: Using static block.
    // static {
    //     count = 10;
    // }

}
```

- 实例成员变量  

必须在`声明该实例成员变量`、`构造器`或`非静态初始化代码块`中指定初始值，并且只能`三者选其一`。  


``` java
public class FinalVariableTest {
    // Method1: Specifying initial value when declaring variables.
    private final int age = 24;

    // Method2: Using constructor.
    // public FinalVariableTest(int age){
    //     this.age = age;
    // }

    // Method3: Using statement block.
    // {
    //     age = 20;
    // }
}
```


## `final`局部变量  

若`final`修饰的局部变量在定义时未指定默认值，则可以在后面代码中对该`final`变量赋初始值，但只能一次，不能重复赋值。  

``` java
final int a = 1;
final int b;

// Cannot assign a value to final variable a.
// a = 2;
b = 2;
```

再来看个有意思的地方：  

``` java
public void test(final int a){
    a = 10;
}
```

想一下，以上方法是否可以编译通过？  

不能的原因是什么？  

因为当`test()`方法被调用时，形参`a`会根据传入的`实参`进行初始化，所以被`final`修饰的变量`a`，无法进行第二次赋值`a = 10;`。  


## `final`修饰基本类型变量与引用类型变量的区别  

使用`final`修饰基本类型变量时，不能对其重复赋值，但对于引用类型变量而言，它保存的仅仅是一个引用，`final`只能保证该引用变量的引用地址不会发生改变，但这个对象的内部完全可以改变。  

``` java
@Test
public void referenceFinalTest(){

    final String[] names = new String[10];
    // The reference of names is [Ljava.lang.String;@11bb3ab
    System.out.println("The reference of names is " + names);

    names[0] = "frankie";
    // The reference of names is [Ljava.lang.String;@11bb3ab
    System.out.println("The reference of names is " + names);

    // 0th element in the names is frankie
    System.out.println("0th element in the names is " + names[0]);
}
```

## `Reference`  
- 《疯狂Java讲义 - 6.4》 - `final修饰符`  