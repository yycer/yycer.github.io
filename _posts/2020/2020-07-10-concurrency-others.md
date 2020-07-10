---
layout: post
title: 并发 - [11] 其他内容
category: multithread
tags: [multithread]
excerpt: Concurreny Others
---

以下内容总结自：   
  
`徐隆曦 - Java 并发编程 78 讲 - 剩余章节`    



## CAS  

什么是`CAS`操作？  

`CAS`的全称是`Compare And Swap`。  

它的作用是将`变量a`在`内存中的值`与`指定值`进行比较，若相等，  

则将`a`的值修改为`新值`，否则操作失败，整个过程具有`原子性`。   

那么`CAS`有什么缺点？  

第一，`ABA`问题。  

举个例子，假设现在`变量a`在内存中为`10`，然后线程`A`执行如下操作：  

`compareAndSwap(this, 10, 20)`，执行完成后，内存中值变成了`20`。   

然后，线程`B`执行如下操作：  

`compareAndSwap(this, 20, 10)`，又将变量`a`设置回了`10`。  

那么对于线程`A`来说就会产生疑惑，虽然还是`10`，但是从内存的角度，  

其实经历了如下变化过程： `10 -> 20 -> 10`  

那么如何解决这个问题？  

加个`版本号`就行，比如：原子类中的`AtomicStampedReference`。  

所以，`CAS`检查的并不是值有没有发生过变化，而是比较当前值与预期值是否相等。  


第二，自旋时间过长，会浪费大量`CPU`等资源。  



## 死锁    

举个例子：  

``` java
public class DeadLockDemo {

    private static Object o1 = new Object();
    private static Object o2 = new Object();

    public static void main(String[] args) throws InterruptedException {

        Runnable task1 = () -> {
            synchronized (o1) {
                System.out.println(Thread.currentThread().getName() + "获取o1");
                try {
                    Thread.sleep(3000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                synchronized (o2) {
                    System.out.println(Thread.currentThread().getName() + "获取o2");
                }
            }
        };

        Runnable task2 = () -> {
            synchronized (o2) {
                System.out.println(Thread.currentThread().getName() + "获取o2");
                try {
                    Thread.sleep(2000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                synchronized (o1) {
                    System.out.println(Thread.currentThread().getName() + "获取o1");
                }
            }
        };

        Thread t1 = new Thread(task1, "Thread1");
        Thread t2 = new Thread(task2, "Thread2");
        t1.start();
        t2.start();
        t1.join();
        t2.join();
    }
}
```

如何使用命令行查看死锁信息？  

首先，执行`jsp`命令，查看程序的`进程号`：  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/jps_commands.png)  

然后，执行`jstack processId`命令：  



![](https://yyc-images.oss-cn-beijing.aliyuncs.com/deadlock_info.png)  


###  死锁的四个必要条件：  

第一，独占  

一把锁每次只能被一个线程所占有。  

第二，不释放   

一个线程因请求其他锁而发生阻塞时，不释放已有锁。  

第三，不剥夺  

当前线程获取的锁，在没使用完之前，不会被强行剥夺。  

第四，成环  

线程间的请求资源关系成环。  

### 如何解决死锁问题？  

第一，避免策略  

注意加锁和解锁的顺序。  

第二，检测和恢复策略。  

定时检测程序中锁的链路情况，防止出现环，并及时选择恢复策略，  

如释放某个线程持有的资源、中断某个线程、重启等。  


第三，鸵鸟策略  

如果当前系统的并发程度并不高，或者发生死锁的情况非常少，如：内部系统、管理系统。  

则可以暂时不用考虑死锁场景。  




## String的不变特性的好处？  

主要有如下三点：  

第一，可以构建`字符串常量池`，避免创建相同的字符串。  

第二，可以作为`HashMap`的`key`。  

第三，保证线程安全。    



