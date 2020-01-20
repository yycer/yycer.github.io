---
layout: post
title: 多线程 - ReentrantReadWriteLock
category: multithread
tags: [multithread]
excerpt: ReentrantReadWriteLock
---

首先，为什么要使用`ReentrantReadWriteLock`？   

我的理解是：由于`ReentrantLock`具有完全互斥排他的效果，虽然保证了线程安全，但是效率非常低，因此在多线程场景中，当你不需要处理共享数据时，你就需要一种更高效的解决方案。  

然后，读写锁的重点是什么？  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/read_write_lock.png)

下面实践一番。  

``` java
public class PlayReentrantReadWriteLock {

    ReentrantReadWriteLock lock = new ReentrantReadWriteLock();

    public void read(){
        try {
            lock.readLock().lock();
            System.out.println(LocalDateTime.now() + " [" + Thread.currentThread().getName() + "] "
                + "enter read()");
            Thread.sleep(3000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            lock.readLock().unlock();
        }
    }

    public void write(){
        try {
            lock.writeLock().lock();
            System.out.println(LocalDateTime.now() + " [" + Thread.currentThread().getName() + "] "
                    + "enter write()");
            Thread.sleep(3000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            lock.writeLock().unlock();
        }
    }
}

```




## `Read-Read`  

``` java
@Test
public void readReadTest(){
    PlayReentrantReadWriteLock rrwLock = new PlayReentrantReadWriteLock();
    new Thread(() -> rrwLock.read(), "threadA").start();
    new Thread(() -> rrwLock.read(), "threadB").start();

//        2020-01-20T10:51:29.877 [threadB] enter read()
//        2020-01-20T10:51:29.877 [threadA] enter read()
}
```

## `Read-Write`  

``` java
@Test
public void readWriteTest() throws InterruptedException {
    PlayReentrantReadWriteLock rrwLock = new PlayReentrantReadWriteLock();
    new Thread(() -> rrwLock.read(), "threadA").start();
    new Thread(() -> rrwLock.write(), "threadB").start();
    Thread.sleep(3100);

//        2020-01-20T10:51:02.387 [threadA] enter read()
//        2020-01-20T10:51:05.388 [threadB] enter write()
}

```

## `Write-Read`  

``` java
@Test
public void writeReadTest() throws InterruptedException {
    PlayReentrantReadWriteLock rrwLock = new PlayReentrantReadWriteLock();
    new Thread(() -> rrwLock.write(), "threadA").start();
    new Thread(() -> rrwLock.read(), "threadB").start();
    Thread.sleep(3100);

//        2020-01-20T10:52:16.997 [threadA] enter write()
//        2020-01-20T10:52:19.997 [threadB] enter read()
}
```

## `Write-Write`  

``` java
@Test
public void writeWriteTest() throws InterruptedException {
    PlayReentrantReadWriteLock rrwLock = new PlayReentrantReadWriteLock();
    new Thread(() -> rrwLock.write(), "threadA").start();
    new Thread(() -> rrwLock.write(), "threadB").start();
    Thread.sleep(3100);

//        2020-01-20T10:52:59.941 [threadA] enter write()
//        2020-01-20T10:53:02.943 [threadB] enter write()
}
```

- `《Java多线程编程核心技术》 - 4.1 使用ReentrantReadWriteLock类`  
