---
layout: post
title: Thread中易混淆方法总结
category: multithread
tags: [java, basic-java]
excerpt: sleep()、yiled()、wait()、notify()、join()方法总结
---
![overview](http://px8rn4o1y.bkt.clouddn.com/thread_confused_methods.png)
<br>
<br>
## Thread.sleep()  

- 方法类型: `Thread`类中的静态方法。
- 方法特点: 释放当前线程的CPU资源，但不释放锁资源`Monitor`。

```java
@Test
public void sleepTest() throws InterruptedException {
    Object object = new Object();

    Thread threadA = new Thread(() -> {
        synchronized (object){
            try {
                log.warn("ThreadA start!");
                Thread.sleep(100);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    });

    Thread threadB = new Thread(() -> {
        synchronized (object){
           log.warn("ThreadB start!");
        }
    });

    threadA.start();
    // Guaranteed threadA run first.
    Thread.sleep(3);
    // Since threadA call the sleep() method, the object is not released,  
    // so threadB has no chance to execute.
    threadB.start();

    while (true){
        try {
            Thread.sleep(30);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        log.warn("The state of threadA: " + threadA.getState());
        if (Thread.State.TERMINATED.equals(threadA.getState())){
            log.warn("The state of threadA: " + threadA.getState());
            break;
        }
    }
}
```

> 执行结果  

![sleep_result](http://px8rn4o1y.bkt.clouddn.com/sleep_result.png)
<br>
<br>

## Thread.yield()  
- 方法类型: `Thread`类的静态方法。
- 方法特点: 当前线程让出时间片`timeslice`，优先让其他线程执行。

```java
@Test
public void yieldTest(){
    Thread threadA = new Thread(() -> {
        log.warn(Thread.currentThread().getName() + " : 开始执行!");
        try {
            Thread.sleep(1);
            log.warn(Thread.currentThread().getName() + " : 执行结束!");
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }, "threadA");

    threadA.start();

    Thread threadB = new Thread(() -> {
        while (!Thread.State.TERMINATED.equals(threadA.getState())){
            // threadB let threadA execute first.
            Thread.yield();
            log.warn(Thread.currentThread().getName() + " : 我在等threadA执行结束!");
        }
        log.warn(Thread.currentThread().getName() + " : 执行结束!");
    }, "threadB");
    threadB.start();
}
```
> 执行结果  
  
![yield_result](http://px8rn4o1y.bkt.clouddn.com/yield_result.png)
<br>
<br>
## Object.wait() & Object.notify()  
- 方法类型: `wait()`和`notify()`都是`Object`的实例方法。
- 方法特点: 
    - `wait()`必须在`同步方法`、`同步块`中执行。
    - 释放当前线程的CPU资源和锁资源`Monitor`。
    - 会持续阻塞，除非使用`notify()`、`notifyAll()`方法显式唤醒。  

> 一道有意思的题目: 创建两个线程，交错打印1~10.  


```java
@Test
public void waitAndNotifyTest() throws InterruptedException {
    Object object = new Object();

    Thread threadA = new Thread(() -> {
        synchronized (object){
            for (int i = 0; i <= 10; i += 2){
                log.warn("i = " + i);
                object.notify();
                try {
                    object.wait();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }
    }, "threadA");

    Thread threadB = new Thread(() -> {
        synchronized (object){
            for (int i = 1; i < 10; i += 2){
                log.warn("i = " + i);
                object.notify();
                try {
                    object.wait();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }
    }, "threadB");

    threadA.start();
    Thread.sleep(3);
    threadB.start();
}
```

> 执行结果  
  
![wait_and_notify_result](http://px8rn4o1y.bkt.clouddn.com/wait_and_notify_result.png)
<br>
<br>

## thread.join()  
- 方法类型: `Thread`实例方法。
- 方法特点: 保证先将线程A中逻辑执行完毕，然后再执行线程B中逻辑。


```java
@Test
public void joinTest() throws InterruptedException {
    log.warn(Thread.currentThread().getName() + " : 开始执行!");

    Thread threadA = new Thread(() -> {
        for (int i = 0; i < 5; i++){
            try {
                Thread.sleep(100);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            log.warn(Thread.currentThread().getName() + " : 执行中!");
        }
    });

    threadA.start();
    Thread.sleep(200);
    threadA.join();
    log.warn(Thread.currentThread().getName() + " : 执行完毕!");
}
```

> 执行结果 - 包含join()  

![with_join_result](http://px8rn4o1y.bkt.clouddn.com/with_join_result.png)

> 执行结果 - 不包含join()  
  
![without_join_result](http://px8rn4o1y.bkt.clouddn.com/without_join_result.png)
<br>
<br>
## 参考文章
- [五月的仓颉 - 多线程系列](https://www.cnblogs.com/xrq730/category/733883.html){:target="_blank"}
- [Java 多线程编程之：notify 和 wait 用法](https://segmentfault.com/a/1190000018096174#articleHeader0){:target="_blank"}
- [sleep、yield、join方法简介与用法 sleep与wait区别 多线程中篇（十五）](https://www.cnblogs.com/noteless/p/10443446.html){:target="_blank"}