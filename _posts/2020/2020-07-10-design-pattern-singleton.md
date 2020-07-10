---
layout: post
title: 设计模式 - 单例
category: design-pattern
tags: [design-pattern]
excerpt: Singleton
---

以下内容总结自：   
  
`徐隆曦 - 在Java中如何写一个正确的单例模式？`  

## 什么是单例模式？   

一个类只有一个实例，并且提供一个全局可以访问的入口。  


## 为什么需要单例模式？  

第一，节省内存、节省计算。  

第二，便于管理。  

## 单例模式有哪些应用场景？  

- 网站的全局计数器。  
- `Windows`的垃圾回收站、任务管理器。  
- 数据库连接池、线程池。  


## 如何实现单例模式？  

### 饿汉式  

先来看下实现：  

``` java
public class Singleton1 {

    private static Singleton1 singleton1 = new Singleton1();

    public static Singleton1 getInstance(){
        return singleton1;
    }
}
```

再来看下使用静态代码块的方式实现：  

``` java
public class Singleton2 {

    private static Singleton2 singleton2;

    static {
        singleton2 = new Singleton2();
    }

    public static Singleton2 getInstance(){
        return singleton2;
    }
}
```

所谓饿汉式，就是只要类一被加载，则创建实例。  

这样的话，有什么问题呢？  

如果程序自始至终都没使用过这个实例，就会造成内存的浪费。  

### 懒汉式  

为了解决`饿汉式`的问题，于是我们引入`懒汉式`：  

核心思想在于，一旦需要才会创建，属于`延迟`加载。  

来看下实现：  


``` java
public class Singleton3 {

    private static Singleton3 singleton3;

    public static Singleton3 getInstance(){
        if (singleton3 == null){
            singleton3 = new Singleton3();
        }
        return singleton3;
    }
}

```

那么`懒汉式`有什么问题呢？  

一旦在`多线程`环境下，就会重复创建对象。  

那么我们可以通过`synchronzied`关键字来实现同步：  

``` java
public class Singleton4 {

    private static Singleton4 singleton4;

    public static synchronized Singleton4 getInstance(){
        if (singleton4 == null){
            singleton4 = new Singleton4();
        }
        return singleton4;
    }
}
```

上面的锁粒度太大了，我们仅仅需要锁住创建实例的部分：  

``` java
public class Singleton5 {

    private static Singleton5 singleton5;

    public static Singleton5 getInstance(){
        if (singleton5 == null){
            synchronized (Singleton5.class){
                singleton5 = new Singleton5();
            }
        }
        return singleton5;
    }
}
```

### 双重检查式  

上面的写法，虽然降低了锁的粒度，但是在并发激烈的情况下，仍会重复创建对象。  

假设`线程A`和`B`同时执行到`if()`判断语句，此时`线程A`获取`monitor`锁， 

然后创建对象，并释放锁，紧接着，`线程B`仍会创建一个对象。  

所以，我们需要使用`双重if判断`：  


``` java
public class Singleton6 {

    private static volatile Singleton6 singleton6;

    public static Singleton6 getInstance(){
        if (singleton6 == null){
            synchronized (Singleton6.class){
                if (singleton6 == null){
                    singleton6 = new Singleton6();
                }
            }
        }
        return singleton6;
    }
}
```

那么为什么`singleton6`对象需要使用`volatile`关键字修饰？  

目的是防止`重排序`，避免拿到`未完全初始化`的对象。   

`singleton = new Singleton();`  

对于上面的语句，`JVM`最少包含以下三步操作：  

第一步，为`singleton`分配内存空间。  

第二步，调用`Singleton`的构造函数等来初始化`singleton`对象。  

第三步，将`singleton`对象指向分配的内存空间。  

但是，可能存在`重排序`的优化，如：`1-3-2`   

假设`线程A`执行到第三步，此时`singleton`已经不是`null`了，  

然后`线程B`也来获取实例，但此时`singleton`其实并没有`完全初始化`，所以可能会导致异常。  



### 内部静态类式  

再来看下`内部静态类`的实现方式：  

``` java
public class Singleton7 {

    private static class SingletonInstance{
        private static final Singleton7 singleton = new Singleton7();
    }

    public static Singleton7 getInstance(){
        return SingletonInstance.singleton;
    }
}
```

它和`双重检查锁`一样，既保证了`线程安全`，也实现了`延迟加载`。  



### 枚举式  


最后，再来看下`枚举`实现方式：  

``` java
public enum Singleton8 {

    INSTANCE;
    
}
```

它的特点的是什么呢？  


第一，写法简单、优雅。  

第二，通过`JVM`实现了线程安全。  

第三，防止`序列化`、`反射`破坏单例。  

