---
layout: post
title: 多线程 - Synchronized同步代码块
category: multithread
tags: [multithread]
excerpt: Synchronized Statement Block.
---

## `synchronized方法的弊端`  

``` java
public class SyncMethodDisadvantage {

    private String processResult = "Done";

    synchronized public void syncMethod(){
        System.out.println(LocalDateTime.now() + " syncMethod() start in " + Thread.currentThread().getName());
        try {
            Thread.sleep(2000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println(LocalDateTime.now() + " processResult = " + processResult);
        System.out.println(LocalDateTime.now() + " syncMethod() end   in " + Thread.currentThread().getName());
    }
}

@Test
public void syncMethodDisadvantageTest() throws InterruptedException {
    SyncMethodDisadvantage disadvantage = new SyncMethodDisadvantage();
    new Thread(() -> disadvantage.syncMethod(), "threadA").start();
    new Thread(() -> disadvantage.syncMethod(), "threadB").start();
    Thread.sleep(4100);
}

-------------  Console  --------------
// 2020-01-12T18:21:36.973 syncMethod() start in threadA
// 2020-01-12T18:21:38.973 processResult = Done
// 2020-01-12T18:21:38.973 syncMethod() end   in threadA
// 2020-01-12T18:21:38.973 syncMethod() start in threadB
// 2020-01-12T18:21:40.974 processResult = Done
// 2020-01-12T18:21:40.974 syncMethod() end   in threadB

```

上面的例子中关键点在于`processResult`，因此你只要控制它的并发写入或访问就够了。  

``` java
public class SyncMethodDisadvantage {

    private String processResult = "Done";

    public void syncBlock(){
        System.out.println(LocalDateTime.now() + " syncMethod() start in " + Thread.currentThread().getName());
        try {
            Thread.sleep(2000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        synchronized (this){
            System.out.println(LocalDateTime.now() + " processResult = " + processResult);
        }
        System.out.println(LocalDateTime.now() + " syncMethod() end   in " + Thread.currentThread().getName());
    }
}

@Test
public void syncBlockTest() throws InterruptedException {
    SyncMethodDisadvantage disadvantage = new SyncMethodDisadvantage();
    new Thread(() -> disadvantage.syncBlock(), "threadA").start();
    new Thread(() -> disadvantage.syncBlock(), "threadB").start();
    Thread.sleep(4100);
}

-------------  Console  --------------
// 2020-01-12T18:24:29.337 syncMethod() start in threadA
// 2020-01-12T18:24:29.337 syncMethod() start in threadB
// 2020-01-12T18:24:31.337 processResult = Done
// 2020-01-12T18:24:31.338 processResult = Done
// 2020-01-12T18:24:31.338 syncMethod() end   in threadB
// 2020-01-12T18:24:31.338 syncMethod() end   in threadA
```

## `synchronized同步代码块间的同步性`  

当一个线程访问`object`的`synchronized同步代码块`时，其他线程对同一个`object`中所有其他`synchronized同步代码块`的访问都会被阻塞。  


``` java
public class SyncBlockSynchronicity {

    public void syncBlock1(){
        synchronized (this){
            System.out.println(LocalDateTime.now() + " syncBlock1() start in " + Thread.currentThread().getName());
            try {
                Thread.sleep(2000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(LocalDateTime.now() + " syncBlock1() end   in " + Thread.currentThread().getName());
        }
    }

    public void syncBlock2(){
        synchronized (this){
            System.out.println(LocalDateTime.now() + " syncBlock2() start in " + Thread.currentThread().getName());
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(LocalDateTime.now() + " syncBlock2() end   in " + Thread.currentThread().getName());
        }
    }
}

@Test
public void syncBlockSynchronicityTest() throws InterruptedException {
    SyncBlockSynchronicity synchronicity = new SyncBlockSynchronicity();
    new Thread(() -> synchronicity.syncBlock1(), "threadA").start();
    new Thread(() -> synchronicity.syncBlock2(), "threadB").start();
    Thread.sleep(3100);
}

-------------  Console  --------------
// 2020-01-12T18:36:04.202 syncBlock1() start in threadA
// 2020-01-12T18:36:06.202 syncBlock1() end   in threadA
// 2020-01-12T18:36:06.202 syncBlock2() start in threadB
// 2020-01-12T18:36:07.202 syncBlock2() end   in threadB
```

## `静态同步synchronized方法和synchronized(Class)代码块`  

静态同步`synchronized`方法和`synchronized(Class)`代码块锁的是`Class类`，它对相关所有实例都起作用。  


下面分别比较同步方法、同步静态方法、同步静态类代码块的区别：  


``` java
public class DiffBetweenClassAndObjectLock {

    /**
     * sync methods.
     */
    synchronized public void method1(){
        System.out.println(LocalDateTime.now() + " method1() start in " + Thread.currentThread().getName());
        try {
            Thread.sleep(2000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println(LocalDateTime.now() + " method1() end   in " + Thread.currentThread().getName());
    }

    synchronized public void method2(){
        System.out.println(LocalDateTime.now() + " method2() start in " + Thread.currentThread().getName());
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println(LocalDateTime.now() + " method2() end   in " + Thread.currentThread().getName());
    }

    /**
     * sync static methods.
     */
    synchronized public static void staticMethod1(){
        System.out.println(LocalDateTime.now() + " staticMethod1() start in " + Thread.currentThread().getName());
        try {
            Thread.sleep(2000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println(LocalDateTime.now() + " staticMethod1() end   in " + Thread.currentThread().getName());
    }

    synchronized public static void staticMethod2(){
        System.out.println(LocalDateTime.now() + " staticMethod2() start in " + Thread.currentThread().getName());
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println(LocalDateTime.now() + " staticMethod2() end   in " + Thread.currentThread().getName());
    }

    /**
     * sync static blocks.
     */
    public void staticBlockMethod1(){
        synchronized (DiffBetweenClassAndObjectLock.class){
            System.out.println(LocalDateTime.now() + " staticBlockMethod1() start in " + Thread.currentThread().getName());
            try {
                Thread.sleep(2000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(LocalDateTime.now() + " staticBlockMethod1() end   in " + Thread.currentThread().getName());
        }
    }

    public void staticBlockMethod2(){
        synchronized (DiffBetweenClassAndObjectLock.class){
            System.out.println(LocalDateTime.now() + " staticBlockMethod2() start in " + Thread.currentThread().getName());
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(LocalDateTime.now() + " staticBlockMethod2() end   in " + Thread.currentThread().getName());
        }
    }
}

@Test
public void syncSyncMethodTest() throws InterruptedException {

    // Step1: sync method.
    DiffBetweenClassAndObjectLock classLock1 = new DiffBetweenClassAndObjectLock();
    DiffBetweenClassAndObjectLock classLock2 = new DiffBetweenClassAndObjectLock();
    new Thread(() -> classLock1.method1(), "threadA").start();
    new Thread(() -> classLock2.method2(), "threadB").start();
    Thread.sleep(3100);
}

-------------  Console  --------------
// 2020-01-12T19:26:37.609 method2() start in threadB
// 2020-01-12T19:26:37.609 method1() start in threadA
// 2020-01-12T19:26:38.609 method2() end   in threadB
// 2020-01-12T19:26:39.609 method1() end   in threadA

@Test
public void syncSyncMethodTest() throws InterruptedException {

    // Step2: sync static method.
    DiffBetweenClassAndObjectLock classLock3 = new DiffBetweenClassAndObjectLock();
    DiffBetweenClassAndObjectLock classLock4 = new DiffBetweenClassAndObjectLock();
    new Thread(() -> classLock3.staticMethod1(), "threadA").start();
    new Thread(() -> classLock4.staticMethod2(), "threadB").start();
    Thread.sleep(3100);
}

-------------  Console  --------------
// 2020-01-12T19:27:36.450 staticMethod1() start in threadA
// 2020-01-12T19:27:38.451 staticMethod1() end   in threadA
// 2020-01-12T19:27:38.451 staticMethod2() start in threadB
// 2020-01-12T19:27:39.451 staticMethod2() end   in threadB

@Test
public void syncSyncMethodTest() throws InterruptedException {

    // Step3: sync static block.
    DiffBetweenClassAndObjectLock classLock5 = new DiffBetweenClassAndObjectLock();
    DiffBetweenClassAndObjectLock classLock6 = new DiffBetweenClassAndObjectLock();
    new Thread(() -> classLock5.staticBlockMethod1(), "threadA").start();
    new Thread(() -> classLock6.staticBlockMethod2(), "threadB").start();
    Thread.sleep(3100);
}

-------------  Console  --------------
// 2020-01-12T19:28:27.666 staticBlockMethod1() start in threadA
// 2020-01-12T19:28:29.666 staticBlockMethod1() end   in threadA
// 2020-01-12T19:28:29.666 staticBlockMethod2() start in threadB
// 2020-01-12T19:28:30.667 staticBlockMethod2() end   in threadB
```

可以看到同步代码块仍对应那条规则：多个对象多个锁。  
但是，对于静态同步方法或代码块来说，这个类的所有实例用的都是一把锁。  


## 多线程的死锁     

什么是死锁？  

先来看张图：  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/dead_lock.png)

我的理解是：`线程A`已占有`资源A`，但还想要`资源B`，与此同时，`线程B`已占有`资源B`，但也想要`资源A`。`(吃着碗里的，看着对方碗里的)`  


``` java
public class DeadLock {

    public Object o1 = new Object();
    public Object o2 = new Object();

    public void tryDeadLock(String username){
        if ("a".equals(username)){
            synchronized (o1){
                try {
                    System.out.println(
                            "current thread name " + Thread.currentThread().getName() +
                            ", a get o1");
                    Thread.sleep(10000);
                    synchronized (o2){
                        System.out.println(
                                "current thread name " + Thread.currentThread().getName() +
                                ", a get o2");
                    }
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        } else {
            synchronized (o2){
                try {
                    System.out.println(
                            "current thread name " + Thread.currentThread().getName() +
                            ", b get o2");
                    Thread.sleep(15000);
                    synchronized (o1){
                        System.out.println(
                                "current thread name " + Thread.currentThread().getName() +
                                ", b get o1");
                    }
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}

@Test
public void deadLockTest() throws InterruptedException {
    DeadLock deadLock = new DeadLock();
    new Thread(() -> deadLock.tryDeadLock("a"), "threadA").start();
    Thread.sleep(10);
    new Thread(() -> deadLock.tryDeadLock("b"), "threadB").start();
}
```

## 锁对象的改变  

锁对象一旦改变，就变成多个对象，多个锁了。  

``` java

public class LockObjectChange {

    public String lock = "123";

    public void justPrint(){
        synchronized (lock){
            System.out.println(LocalDateTime.now() + " current thread = " + Thread.currentThread().getName());
            lock = "456";
            try {
                Thread.sleep(2000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(LocalDateTime.now() + " current thread = " + Thread.currentThread().getName());
        }
    }
}

@Test
public void changeLockObjectTest() throws InterruptedException {
    LockObjectChange loc = new LockObjectChange();
    new Thread(() -> loc.justPrint(), "threadA").start();
    Thread.sleep(50);
    new Thread(() -> loc.justPrint(), "threadB").start();
    Thread.sleep(4100);
}

-------------  Console  --------------
/**
 * Without lock = "456";
 */
//        2020-01-10T21:27:25.811 current thread = threadA
//        2020-01-10T21:27:27.811 current thread = threadA
//        2020-01-10T21:27:27.811 current thread = threadB
//        2020-01-10T21:27:29.811 current thread = threadB


/**
 * With lock = "456";
 */
//        2020-01-10T21:28:05.925 current thread = threadA
//        2020-01-10T21:28:05.972 current thread = threadB
//        2020-01-10T21:28:07.925 current thread = threadA
//        2020-01-10T21:28:07.973 current thread = threadB
```

但是，只要对象不变，即使改变了对象的属性，仍会同步执行。  


``` java
@Setter
@Getter
public class Order {

    public String orderId;
    public String amount;
}

public class JustChangeProperty {

    public void justChangeProperty(Order order){
        synchronized (order){
            System.out.println(LocalDateTime.now() + " current thread = " + Thread.currentThread().getName());
            try {
                Thread.sleep(2000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(LocalDateTime.now() + " current thread = " + Thread.currentThread().getName());
        }
    }
}

@Test
public void justChangePropertyTest() throws InterruptedException {
    JustChangeProperty jcp = new JustChangeProperty();
    Order order = new Order();
    order.setAmount("20");
    new Thread(() -> jcp.justChangeProperty(order), "threadA").start();
    order.setAmount("50");
    new Thread(() -> jcp.justChangeProperty(order), "threadB").start();
    Thread.sleep(5000);
}

-------------  Console  --------------
//        2020-01-10T21:32:12.949 current thread = threadA
//        2020-01-10T21:32:14.949 current thread = threadA
//        2020-01-10T21:32:14.949 current thread = threadB
//        2020-01-10T21:32:16.951 current thread = threadB
```

## `Reference`  
- `《Java多线程编程核心技术》 - 2.2 synchronized同步语句块`


