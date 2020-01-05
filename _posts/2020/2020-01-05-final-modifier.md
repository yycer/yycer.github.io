---
layout: post
title: Final修饰符
category: Spring
tags: [spring, java]
excerpt: Java Final Modifier
---

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/final.png)

## `final`成员变量  

成员变量是随`类初始化`或`对象初始化`而初始化的。  

- 类成员变量  

必须在声明该类变量或使用静态初始块时指定初始值，并且只能二者选其一。  


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

- 实例变量  

必须在声明该实例变量、使用构造器或非静态初始块时指定初始值，并且只能三者选其一。  


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

## Reference  
- 《疯狂Java讲义 - 6.4》 - `final修饰符`  