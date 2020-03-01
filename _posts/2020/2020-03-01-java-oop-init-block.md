---
layout: post
title: Java - [面向对象]初始化代码块
category: java-basic
tags: [spring, java]
excerpt: Java OOP Init Block
---

## 实例初始化代码块  

首先来看个有意思的问题：`实例初始化代码块`、`初始化实例成员变量`、`构造器初始化`三者的执行顺序如何？  


``` java
@Getter
public class InstanceInit {

    public InstanceInit(){
        this.age = 40;
    }

    {
        age = 10;
    }

    int age = 20;

    {
        age = 30;
    }
}

@Test
void instanceInitTest(){
    InstanceInit ii = new InstanceInit();
    System.out.println("Age is " + ii.getAge());
}
```

老规矩，`age`是多少？  

答案是`40`。

原因是：先执行`实例初始化代码块`或`实例成员变量初始化`，再执行`构造器初始化`。  

而前两者是根据具体的`代码顺序`决定的。  

如下图所示：  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/init_block_class.png)  


再来说下，继承关系下`实例初始化代码块`与`构造器`的执行顺序：  

创建一个Java对象时，不仅会执行该类的`实例初始化代码块`和`构造器`，而且会一直追溯到`java.lang.Object`类，优先执行`Object`类的`实例初始化代码块`和`构造器`，然后顺着父类关系一层层执行，最后才是该类。  

举个例子：  


``` java
public class BaseClass {

    public BaseClass(){
        System.out.println("Constructor in BaseClass.");
    }

    {
        System.out.println("Instance init block in BaseClass.");
    }
}

public class SubClass extends BaseClass{

    public SubClass(){
        System.out.println("Constructor in SubClass.");
    }

    {
        System.out.println("Instance init block in SubClass.");
    }
}

@Test
void instanceInitSeqTest(){
    new SubClass();
}

// Instance init block in BaseClass.
// Constructor in BaseClass.
// Instance init block in SubClass.
// Constructor in SubClass.
```

其实还执行了`Object`类中的默认构造函数，只不过它没有打印出任何信息~  



## 类初始化代码块  


先来看下`类初始化代码块`和`初始化类成员变量`之间的顺序关系：

``` java
public class StaticInit {

    static {
        a = 5;
    }

    public static int a = 10;

    static {
        a = 20;
    }
}

@Test
void staticInitTest(){
    System.out.println("a = " + StaticInit.a); // 20
}
```

实验证明，和实例执行顺序一致。  



再来看下继承关系下，`类初始化代码块`、`实例初始化代码块`与`构造器`之间的执行顺序：  

``` java
public class Root {
    static {
        System.out.println("Root static init block.");
    }

    {
        System.out.println("Root init block.");
    }

    public Root(){
        System.out.println("Root constructor.");
    }
}

public class Mid extends Root{

    static {
        System.out.println("Mid  static init block.");
    }

    {
        System.out.println("Mid  init block.");
    }

    public Mid(){
        System.out.println("Mid  constructor.");
    }

    public Mid(String name){
        this();
        System.out.println(name + " in Mid constructor.");
    }
}

public class Leaf extends Mid{

    static {
        System.out.println("Leaf static init block.");
    }

    {
        System.out.println("Leaf init block.");
    }

    public Leaf(){
        super("frankie");
        System.out.println("Leaf constructor.");
    }
}

@Test
void staticInitBlockTest(){
    new Leaf();
    System.out.println("----------------");
    new Leaf();
}
```

有意思的是第二次创建`Leaf`对象。  

由于`Leaf`类已经初始化成功，所以第二次创建该对象时，无需执行对应的`类初始化代码块`！  


结果如下：  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/static_instance_constructor_init_block.png)  


## `Reference`  
- `《疯狂Java讲义》 - 5.9 初始化块`  
