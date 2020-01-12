---
layout: post
title: 多线程 - Synchronized同步方法
category: multithread
tags: [multithread]
excerpt: Synchronized Method.
---

# 非线程安全  

我们先来看个非线程安全的例子：  

``` java 
public class HashSelfPrivateNum {

    private int num = 0;

    public void addNum(String username) {
        if ("a".equals(username)){
            num = 10;
            System.out.println("a set over.");
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        } else {
            num = 20;
            System.out.println("b set over.");
        }
        System.out.println("username = " + username + ", num = " + num);
    }
}

@Test
public void methodPrivateNumTest() throws InterruptedException {

    HashSelfPrivateNum numRef = new HashSelfPrivateNum();

    Thread threadA = new Thread(() -> numRef.addNum("a"));
    Thread threadB = new Thread(() -> numRef.addNum("b"));
    threadA.start();
    threadB.start();

    Thread.sleep(1500);
}

-------------  Console  --------------
// a set over.
// b set over.
// username = b, num = 20
// username = a, num = 20
```

我们来看下期望和实际得到结果的流程图：  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/thread_unsafety.png)

那么如何获得期望的结果？  

最简单的方法就是将`addNum()`方法设置成`synchronized`的，但这也可能会造成`锁粒度`太粗，导致性能很差。  


# 多个对象多个锁  

``` java 
public class MultiObjectMultiLock {

    synchronized public void methodA(){
        System.out.println("Start methodA at " + LocalDateTime.now() + ", in " + Thread.currentThread().getName());
        try {
            Thread.sleep(2000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("End   methodA at " + LocalDateTime.now() + ", in " + Thread.currentThread().getName());
    }

    synchronized public void methodB(){
        System.out.println("Start methodB at " + LocalDateTime.now() + ", in " + Thread.currentThread().getName());
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("End   methodB at " + LocalDateTime.now() + ", in " + Thread.currentThread().getName());
    }
}

@Test
public void multiObjectMultiLockTest() throws InterruptedException {
    // Be careful, l will create an instance in each thread.
    new Thread(() -> new MultiObjectMultiLock().methodA(), "threadA").start();
    new Thread(() -> new MultiObjectMultiLock().methodB(), "threadB").start();
    Thread.sleep(3200);
}

-------------  Console  --------------
// Start methodB at 2020-01-12T15:01:14.142, in threadB
// Start methodA at 2020-01-12T15:01:14.142, in threadA
// End   methodB at 2020-01-12T15:01:15.142, in threadB
// End   methodA at 2020-01-12T15:01:16.142, in threadA

@Test
public void multiObjectMultiLockTest() throws InterruptedException {
    MultiObjectMultiLock lock = new MultiObjectMultiLock();
    new Thread(() -> lock.methodA(), "threadA").start();
    new Thread(() -> lock.methodB(), "threadB").start();
    Thread.sleep(3200);
}
        
-------------  Console  --------------
// Start methodA at 2020-01-12T15:04:20.966, in threadA
// End   methodA at 2020-01-12T15:04:22.967, in threadA
// Start methodB at 2020-01-12T15:04:22.967, in threadB
// End   methodB at 2020-01-12T15:04:23.967, in threadB
```

可以很清楚的看到，若使用的是同一个对象，执行线程`A`和`B`需要`3`秒，若使用各自创建的对象，就是异步执行，仅需要`2`秒。  


# 一半同步，一半异步  

``` java 
public class HalfAsyncHalfSync {

    synchronized public void syncMethodA(){
        System.out.println("Start sync methodA at " + LocalDateTime.now() + ", in " + Thread.currentThread().getName());
        try {
            Thread.sleep(2000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("End   sync methodA at " + LocalDateTime.now() + ", in " + Thread.currentThread().getName());
    }

    public void methodB(){
        System.out.println("Start methodB at " + LocalDateTime.now() + ", in " + Thread.currentThread().getName());
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("End   methodB at " + LocalDateTime.now() + ", in " + Thread.currentThread().getName());
    }
}

@Test
public void syncMethodHalfSyncHalfAsyncTest() throws InterruptedException {
    HalfAsyncHalfSync half = new HalfAsyncHalfSync();
    new Thread(() -> half.syncMethodA(), "threadA").start();
    new Thread(() -> half.methodB(), "threadB").start();
    Thread.sleep(3200);
}

-------------  Console  --------------
// Start sync methodA at 2020-01-12T15:12:32.266, in threadA
// Start methodB at 2020-01-12T15:12:32.266, in threadB
// End   methodB at 2020-01-12T15:12:33.266, in threadB
// End   sync methodA at 2020-01-12T15:12:34.267, in threadA
```

可以看到`syncMethodA`是同步方法，而`methodB`是一个普通方法，当两者可以异步执行，因此总共花的时间也只有`2`秒。  


# `synchronized重入锁`  

首先解释下什么是重入锁？  

> 自己可以再次获得自己的内部锁。  



``` java 
public class ReentrantService {

    synchronized public void method1(){
        System.out.println(LocalDateTime.now() + "Method1 start");
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        method2();
        System.out.println(LocalDateTime.now() + "Method1 end");
    }

    synchronized private void method2() {
        System.out.println(LocalDateTime.now() + "Method2 start");
        try {
            Thread.sleep(2000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        method3();
        System.out.println(LocalDateTime.now() + "Method2 end");
    }

    synchronized private void method3() {
        System.out.println(LocalDateTime.now() + "Method3 start");
        try {
            Thread.sleep(3000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println(LocalDateTime.now() + "Method3 end");
    }
}

@Test
public void reentrantLockTest() throws InterruptedException {
    // Step1: Call internal nested synchronized methods.
    new Thread(() -> new ReentrantService().method1()).start();
    Thread.sleep(6100);
}

-------------  Console  --------------
// 2020-01-12T15:43:34.918Method1 start
// 2020-01-12T15:43:35.919Method2 start
// 2020-01-12T15:43:37.919Method3 start
// 2020-01-12T15:43:40.919Method3 end
// 2020-01-12T15:43:40.919Method2 end
// 2020-01-12T15:43:40.920Method1 end
```

可重入锁也支持父子类继承的环境中，举个例子：  

``` java 
public class ReentrantChildService extends ReentrantService {

    synchronized public void reentrantChildMethod(){
        System.out.println(LocalDateTime.now() + " reentrantChildMethod() start");
        try {
            Thread.sleep(2000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        this.reentrantParentMethod();
        System.out.println(LocalDateTime.now() + " reentrantChildMethod() end");
    }
}

public class ReentrantService {

    synchronized public void reentrantParentMethod(){
        System.out.println(LocalDateTime.now() + " reentrantParentMethod() start");
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println(LocalDateTime.now() + " reentrantParentMethod() end");
    }
}

@Test
public void reentrantLockTest() throws InterruptedException {

    // Step2: Child class calls synchronized method in parent class.
    new Thread(() -> new ReentrantChildService().reentrantChildMethod()).start();
    Thread.sleep(3100);
}

-------------  Console  --------------
// 2020-01-12T15:53:43.102 reentrantChildMethod() start
// 2020-01-12T15:53:45.102 reentrantParentMethod() start
// 2020-01-12T15:53:46.102 reentrantParentMethod() end
// 2020-01-12T15:53:46.102 reentrantChildMethod() end
```

# 出现异常，锁自动释放 

``` java 
public class ExceptionReleaseLock {
    synchronized public void releaseALock(){
        if ("threadA".equals(Thread.currentThread().getName())){
            for (int i = 0; i < 1000000; i++){
                if (i == 999999){
                    System.out.println(
                            "Current thread name " + Thread.currentThread().getName() +
                             " run time " + LocalDateTime.now());
                    Integer.parseInt("a");
                }
            }
        } else {
            System.out.println(
                    "Current thread name " + Thread.currentThread().getName() +
                    " run time " + LocalDateTime.now());
        }
    }
}

@Test
public void exceptionReleaseLockTest() throws InterruptedException {
    System.out.println(LocalDateTime.now() + " exceptionReleaseLockTest() start.");
    ExceptionReleaseLock exceptionLock = new ExceptionReleaseLock();
    new Thread(() -> exceptionLock.releaseALock(), "threadA").start();
    Thread.sleep(10);
    new Thread(() -> exceptionLock.releaseALock(), "threadB").start();
    Thread.sleep(100);
}

-------------  Console  --------------
// 2020-01-12T15:57:56.289 exceptionReleaseLockTest() start.
// Current thread name threadA run time 2020-01-12T15:57:56.292
// Exception in thread "threadA" java.lang.NumberFormatException: For input string: "a"
//     at java.lang.NumberFormatException.forInputString(NumberFormatException.java:65)
//     at java.lang.Integer.parseInt(Integer.java:580)
//     at java.lang.Integer.parseInt(Integer.java:615)
//     at com.frankie.demo.core_tech.ExceptionReleaseLock.releaseALock(ExceptionReleaseLock.java:17)
//     at com.frankie.demo.coreTechTest.lambda$exceptionReleaseLockTest$8(coreTechTest.java:81)
//     at java.lang.Thread.run(Thread.java:748)
// Current thread name threadB run time 2020-01-12T15:57:56.300
```

当`线程A`遇到异常：`Integer.parseInt("a");`时，就会释放`exceptionLock`这个对象锁，此时`线程B`的机会就来了！  



# 同步不具备继承性  

``` java 
public class ReentrantChildService extends ReentrantService {

    public void childMethod(){
        System.out.println(LocalDateTime.now() + " childMethod() start in " + Thread.currentThread().getName());
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println(LocalDateTime.now() + " childMethod() end in " + Thread.currentThread().getName());
        this.parentSyncMethod();
    }
}

public class ReentrantService {

    synchronized public void parentSyncMethod(){
        System.out.println(LocalDateTime.now() + " parentSyncMethod() start in " + Thread.currentThread().getName());
        try {
            Thread.sleep(2000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println(LocalDateTime.now() + " parentSyncMethod() end in " + Thread.currentThread().getName());
    }
}

@Test
public void syncMethodDoesNotHaveInheritTest() throws InterruptedException {
    ReentrantChildService child = new ReentrantChildService();
    new Thread(() -> child.childMethod(), "threadA").start();
    new Thread(() -> child.childMethod(), "threadB").start();
    Thread.sleep(7100);
}

-------------  Console  --------------
// 2020-01-12T16:10:23.460 childMethod() start in threadB
// 2020-01-12T16:10:23.460 childMethod() start in threadA
// 2020-01-12T16:10:24.460 childMethod() end in threadB
// 2020-01-12T16:10:24.460 childMethod() end in threadA
// 2020-01-12T16:10:24.460 parentSyncMethod() start in threadB
// 2020-01-12T16:10:26.460 parentSyncMethod() end in threadB
// 2020-01-12T16:10:26.460 parentSyncMethod() start in threadA
// 2020-01-12T16:10:28.461 parentSyncMethod() end in threadA

/**
 * If we change childMethod to synchronized.
 */
-------------  Console  --------------
// 2020-01-12T16:11:56.762 childMethod() start in threadA
// 2020-01-12T16:11:57.762 childMethod() end in threadA
// 2020-01-12T16:11:57.762 parentSyncMethod() start in threadA
// 2020-01-12T16:11:59.763 parentSyncMethod() end in threadA
// 2020-01-12T16:11:59.763 childMethod() start in threadB
// 2020-01-12T16:12:00.763 childMethod() end in threadB
// 2020-01-12T16:12:00.763 parentSyncMethod() start in threadB
// 2020-01-12T16:12:02.763 parentSyncMethod() end in threadB
```


# `Reference`  
- `《Java多线程编程核心技术》 - 2.1 synchronized同步方法`  
