---
layout: post
title: Java - [面向对象]多态
category: java-basic
tags: [spring, java]
excerpt: Java OOP Polymorphism
---


本文整理下关于多态的几个有意思的地方。  


## 多态  

第一个问题： 多态是什么？  

`相同类型的变量，调用同一个方法呈现出不同的行为。`  

``` java
public class BaseClass {

    public int book = 5;

    public void base(){
        System.out.println("base() in BaseClass.");
    }

    public void test(){
        System.out.println("test() in BaseClass.");
    }
}

public class SubClass extends BaseClass{

    public String book = "Java book";

    public void sub(){
        System.out.println("sub()  in SubClass.");
    }

    @Override
    public void test(){
        System.out.println("test() in SubClass.");
    }
}

@Test
void polymorphismTest(){
    BaseClass polymorphism = new SubClass();
    polymorphism.test();
    polymorphism.base();
    System.out.println("polymorphism book is " + polymorphism.book);
    // Cannot resolve method 'sub' in BaseClass.
    // polymorphism.sub();
}
```

返回什么？  

``` java
// test() in SubClass.
// base() in BaseClass.
// polymorphism book is 5
```

依次来解释下：  

`BaseClass polymorphism = new SubClass();`

首先，`polymorphism`编译阶段的类型是`BaseClass`，而真正运行阶段的类型是`SubClass`。  

引用变量，也就是`polymorphism`，在编译阶段只能调用其编译阶段类型的所有方法，也就是`BaseClass`的所有方法，所以`polymorphism.sub();`无法通过编译。但运行阶段则执行其运行阶段类型所具有的方法，也就是`SubClass`中的方法。  

所以`polymorphism.test();`返回`test() in SubClass.`。  

注意喔，多态发生咯！  

`相同类型(BaseClass)的变量，调用同一个方法(test)，返回的结果不同`。  

``` java
@Test
void polymorphismTest(){
    BaseClass polymorphism = new SubClass();
    polymorphism.test(); // test() in SubClass.

    BaseClass base = new BaseClass();
    base.test(); // test() in BaseClass.
}
```

继续分析：  


而`polymorphism.base();`，运行阶段类型`SubClass`没有`base()`这个方法，所以它会去父类寻找有没有这个方法，`bingo`，因此返回`base() in BaseClass.`。  

还剩最后一个问题，为什么`book`是`5`？而不是`SubClass`类中的`Java book`？  

`通过引用变量去访问它的类实例成员时，系统总是去访问它编译阶段类型所定义的类实例变量。`  


所以`polymorphism.book`的返回是`5`。  



## 类型转换  

先介绍下`instanceof`运算符的用法：

`a instanceof b`  

- 判断`对象a`是否为`类型b`或者其`子类`。   
- 判断`实现类a`是否实现了`接口b`。  

建议先使用`instanceof`运算符，再使用`类型转换运算符`。  

举个例子：  

``` java
@Test
void typeConversionTest(){
    String name = "";
    Object obj = "yyc";
    if (obj instanceof String) {
        name = (String) obj;
    }

    System.out.println("name is " + name); // name is yyc
}
```

## `Reference`  
- `《疯狂Java讲义》 - 5.7 多态`  
