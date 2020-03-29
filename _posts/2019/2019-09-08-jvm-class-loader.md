---
layout: post
title: JVM - 类加载机制
category: jvm
tags: [jvm]
excerpt: JVM Class Loader
---


## `2020-03-29更新`  

参考自[两道面试题，带你解析Java类加载机制](https://www.cnblogs.com/chanshuyi/p/the_java_class_load_mechamism.html){:target="_blank"}

## 类加载机制三大步骤  

> 1.加载 - `Loading`  

- 将什么东西加载到哪里？  

`将.class文件加载到JVM内存中，并在方法区创建对应的Class对象。`  

- `Class`对象的结构是怎么样？  

`包含：魔数、主次版本号、常量池、父类信息及接口、字段、方法、属性等。`  

- 如何获得`.class`文件？  

`编译.java文件生成.class文件`。  


> 2.连接 - `Linking`  

- 校验 - `Verification`  
    + 主要分为四个阶段: 
        + 第一，Class文件格式校验，例如: 检查魔数、主次版本号。
        + 第二，元数据校验，主要查看是否有父类等。
        + 第三，字节码校验，例如: 将一个子类对象赋值给父类数据类型，JVM认为这是安全的。
        + 第四，符号引用校验，检查是否能通过全限定名找到对应Class类。

    + 作用？ 确保.Class文件的字节流数据符合当前虚拟机的要求，并保证虚拟机的安全。  

- 准备 - `Preparation`  
    - 作用？ - 为`类成员变量`在`方法区`分配内存，并设置`零值`。

``` java
public static int static_variable = 1;
public static final int static_final_variable = 2;
```

注意，准备阶段过后，`static_variable`的值为`0`，而`static_final_variable`的值为`2`。  

- 解析 - `Resoultion`  
    - 作用？ 虚拟机将方法区中常量池的`符号引用`替换为`直接引用`。  

> 3.初始化 - `Initialization`  

- 什么时候触发？  

主要分为以下四种：  

- 使用`new`、`getstatic`、`putstatic`和`invokestatic`指令。  
- `父类优先原则`：当初始化一个类时，若其父类尚未初始化，则先初始化其父类。  
- 进行反射调用时。  
- `主类优先原则`：包含`main`方法的类优先初始化。  


举个例子：  

``` java
public class Parent {

    static int staticVar = 10;
    int instanceVar = 20;

    public Parent(){
        System.out.println("Execute constructor in Parent class.");
    }

    static {
        System.out.println("Execute static block in Parent class.");
    }

    {
        System.out.println("Execute common block in Parent class.");
    }

    static void staticMethod(){
        System.out.println("Execute staticMethod() in Parent class.");
    }

    public void instanceMethod(){
        System.out.println("Execute instanceMethod() in Parent class.");
    }

    public static void main(String[] args) {
        System.out.println("Execute main() in Parent class.");
    }
}

public class Child {

    public static void main(String[] args) throws NoSuchMethodException,
            IllegalAccessException, InvocationTargetException,
            InstantiationException {
        /** 1.1. new **/
        new Parent();

//        Execute static block in Parent class.
//        Execute common block in Parent class.
//        Execute constructor in Parent class.

        /** 1.2. getStatic **/
        int ret = Parent.staticVar;
//        Execute static block in Parent class.

        /** 1.3. putStatic **/
        Parent.staticVar = 2;
//        Execute static block in Parent class.

        /** 1.4. invokeStatic **/
        Parent.staticMethod();
//        Execute static block in Parent class.
//        Execute staticMethod() in Parent class.

        /** 2. Parent first **/
        new Child();
//        Execute static block in Parent class.
//        Execute common block in Parent class.
//        Execute constructor in Parent class.

        /** 3. Reflection **/
        Class<Parent> parentClass = Parent.class;
        Parent parent = parentClass.getConstructor().newInstance();
        parent.instanceMethod();

//        Execute static block in Parent class.
//        Execute common block in Parent class.
//        Execute constructor in Parent class.
//        Execute instanceMethod() in Parent class.
    }
}
```


- 初始化过程？  

分为两步：  

- 类初始化：静态代码块、静态赋值语句。  
- 实例初始化：普通代码块、普通赋值语句和构造器。  

举个例子：  

``` java
public class Initialization {

    int instanceVar = 5;
    static int staticVar = 10;

    {
        System.out.println("Execute common block: instanceVar = " + instanceVar);
    }

    static {
        staticVar = 20;
        System.out.println("Execute static block: staticVar = " + staticVar);
    }

    public Initialization() {
        System.out.println("Execute constructor in Initialization.");
    }

    public static void main(String[] args) {
        System.out.println("Execute main() in Initialization.");
        new Initialization();
    }
}

// Execute static block: staticVar = 20
// Execute main() in Initialization.
// Execute common block: instanceVar = 5
// Execute constructor in Initialization.
```



<br>

## 类加载器分类
- 引导类加载器（Bootstrap ClassLoader）
- 扩展类加载器（Extension ClassLoader）
- 系统类加载器（System ClassLoader）
- 用户类加载器（User ClassLoader）
 
<br>
## 疑问
> 1.什么时候触发类加载的加载阶段？  

- 运行时，用到四大字节码指令
    * `new`关键字实例化对象。
    * 读取`getStatic`、设置`putStatic`类字段。
    * 调用类方法`invokeStatic`。
- 对类进行反射调用。
    - 调用`java.lang.reflect`包中方法。
- 父类尚未加载。
    - 当初始化某个类时，若发现该父类尚未初始化，则优先触发父类初始化。
 

> 2.Java虚拟机规范允许系统预加载某些类。  

- 哪些类？ 系统经常用到的，如：`java.util.*`、`java.lang.*`、`sun.reflect.*`
- 如何证明？ 启动项目时添加`-XX:+TraceClassLoading`虚拟机参数。  


![pre-load-class](https://yyc-images.oss-cn-beijing.aliyuncs.com/%E9%A2%84%E5%8A%A0%E8%BD%BD%E7%B1%BB.png)
 
<br>
## 参考
- 本文中，有些细节没有展开论述，原因是发现一篇很好的文章 [Java虚拟机9：Java类加载机制](https://www.cnblogs.com/xrq730/p/4844915.html){:target="_blank"} 里面已经讲得很清楚了，我也就没有必要再做赘述。

- 《深入理解Java虚拟机》 - 第七章(虚拟机类加载机制)
- 《疯狂Java讲义》 - 第十八章(类加载机制与反射)