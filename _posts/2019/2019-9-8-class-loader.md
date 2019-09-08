---
layout: post
title: Java虚拟机类加载机制
category: jvm
tags: [jvm]
excerpt: 类加载机制浅析
---
## 类加载机制三大步骤  

> 1.加载  

- 获得二进制流：获得.Class文件的字节流，获得方式灵活多变，如：zip包、网络、运行时计算生成等。
- 数据类型转换：将第一步获得的字节流以一定的数据结构（魔数、主次版本号、访问权限、常量池、字段表、方法表、属性表等）保存至`方法区`。
- 生成Class对象：在内存中生成一个代表该类的`java.lang.Class`对象。该类一般情况下是在堆中生成的，但对于`HotSpot`虚拟机，则保存在`方法区`中。  

> 2.连接  

- 校验
    + 主要分为四个阶段: 
        + 第一，Class文件格式校验，例如: 检查魔数、主次版本号。
        + 第二，元数据校验，主要查看是否有父类等。
        + 第三，字节码校验，例如: 将一个子类对象赋值给父类数据类型，JVM认为这是安全的。
        + 第四，符号引用校验，检查是否能通过全限定名找到对应Class类。

    + 作用？ 确保.Class文件的字节流数据符合当前虚拟机的要求，并保证虚拟机的安全。  

- 准备
    - 作用？ - 为`类变量`在`方法区`分配内存，并设置`零值`。
- 解析
    - 作用？ 虚拟机将方法区中常量池的`符号引用`替换为`直接引用`。  

> 3.初始化  

- 作用？ 将用户指定的值赋给类变量，以及执行静态代码块。
- 类变量赋值与静态代码块赋值顺序？ 顺序执行，后面的覆盖前面的！  

```java
public class ClassLoaderSequence {

    public static int id = 1;

    static {
        age = 10;
        id  = 2;
    }

    public static int age = 20;
}

@Test
public void staticExcSequenceTest(){
    System.out.println("id  = " + ClassLoaderSequence.id);  // 2
    System.out.println("age = " + ClassLoaderSequence.age); // 20
}
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


![pre-load-class](http://px8rn4o1y.bkt.clouddn.com/%E9%A2%84%E5%8A%A0%E8%BD%BD%E7%B1%BB.png)
 
<br>
## 参考
- 本文中，有些细节没有展开论述，原因是发现一篇很好的文章 [Java虚拟机9：Java类加载机制](https://www.cnblogs.com/xrq730/p/4844915.html){:target="_blank"} 里面已经讲得很清楚了，我也就没有必要再做赘述。

- 《深入理解Java虚拟机》 - 第七章(虚拟机类加载机制)
- 《疯狂Java讲义》 - 第十八章(类加载机制与反射)