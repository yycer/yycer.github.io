---
layout: post
title: 多线程 - ReentrantLock常用方法
category: multithread
tags: [multithread]
excerpt: ReentrantLock Commonly Used Mehtods.
---


## 多线程场景下`ReentrantLock`的执行顺序  

先来回顾下多线程场景`synchronized`代码块的执行顺序：  

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
// 2020-01-19T21:39:48.041 syncBlock1() start in threadA
// 2020-01-19T21:39:50.041 syncBlock1() end   in threadA
// 2020-01-19T21:39:50.041 syncBlock2() start in threadB
// 2020-01-19T21:39:51.042 syncBlock2() end   in threadB
```

`ReentrantLock`执行顺序与同步代码块一致，只是使用方式更丰富：  

``` java
public class PlayLock {
    public void lockMethod1(){
        try {
            lock.lock();
            System.out.println(LocalDateTime.now() + " ["  + Thread.currentThread().getName() +
                    "] enter lockMethod1()");
            Thread.sleep(1000);
            System.out.println(LocalDateTime.now() + " ["  + Thread.currentThread().getName() +
                    "] enter lockMethod1()");
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }

    public void lockMethod2(){
        try {
            lock.lock();
            System.out.println(LocalDateTime.now() + " ["  + Thread.currentThread().getName() +
                    "] enter lockMethod2()");
            Thread.sleep(2000);
            System.out.println(LocalDateTime.now() + " ["  + Thread.currentThread().getName() +
                    "] enter lockMethod2()");
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }
}

@Test
public void reentrantLockExecSeqTest() throws InterruptedException {
    /**
     * Two threads & two methods.
     */
    PlayLock playLock = new PlayLock();
    new Thread(() -> playLock.lockMethod1(), "threadA").start();
    new Thread(() -> playLock.lockMethod2(), "threadB").start();
    Thread.sleep(3100);

    // 2020-01-19T21:42:49.207 [threadA] enter lockMethod1()
    // 2020-01-19T21:42:50.208 [threadA] enter lockMethod1()
    // 2020-01-19T21:42:50.208 [threadB] enter lockMethod2()
    // 2020-01-19T21:42:52.208 [threadB] enter lockMethod2()


    /**
     * Two threads & one methods.
     */
    PlayLock playLock = new PlayLock();
    new Thread(() -> playLock.lockMethod1(), "threadA").start();
    new Thread(() -> playLock.lockMethod1(), "threadB").start();
    Thread.sleep(3100);

    // 2020-01-19T21:44:26.861 [threadA] enter lockMethod1()
    // 2020-01-19T21:44:27.862 [threadA] enter lockMethod1()
    // 2020-01-19T21:44:27.862 [threadB] enter lockMethod1()
    // 2020-01-19T21:44:28.862 [threadB] enter lockMethod1()
}
```


## `getHoldCount()`  

> Queries the number of holds on this lock by the current thread.  

也就是，查询当前线程锁定该锁的次数。  


``` java
public class PlayLock {
    public void holdLockMethod1(){
        try {
            lock.lock();
            System.out.println("Enter holdLockMethod1(), lock count = " + lock.getHoldCount());
            holdLockMethod2();
        } finally {
            lock.unlock();
        }
    }

    private void holdLockMethod2() {
        try {
            lock.lock();
            System.out.println("Enter holdLockMethod2(), lock count = " + lock.getHoldCount());
        } finally {
            lock.unlock();
        }
    }
}

@Test
public void getHoldCountTest(){

    PlayLock playLock = new PlayLock();
    new Thread(() -> playLock.holdLockMethod1()).start();
}

-------------  Console  --------------
   // Enter holdLockMethod1(), lock count = 1
   // Enter holdLockMethod2(), lock count = 2
```


## `QueuedThread`相关方法  

主要涉及如下三个方法：  

> getQueueLength(): Returns an estimate of the number of threads waiting to acquire this lock.  
> hasQueuedThread(Thread thread): Queries whether the given thread is waiting to acquire this lock.  
> hasQueuedThreads(): Queries whether any threads are waiting to acquire this lock.  


``` java
public class PlayLock {

    /**
     * getQueueLength
     */
    public void doGetQueueLength(){
        try {
            lock.lock();
            System.out.println(Thread.currentThread().getName() + " enter doGetQueueLength()");
            Thread.sleep(3000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }

    /**
     * hasQueuedThread()
     * hasQueuedThreads()
     */
    public void doHasQueuedThread(){
        try {
            lock.lock();
            System.out.println(Thread.currentThread().getName() + " enter doHasQueuedThread()");
            Thread.sleep(10000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }
}

@Test
public void getQueueLengthTest() throws InterruptedException {
    PlayLock playLock = new PlayLock();
    new Thread(() -> playLock.doGetQueueLength(), "threadA").start();
    Thread.sleep(20);
    new Thread(() -> playLock.doGetQueueLength(), "threadB").start();
    new Thread(() -> playLock.doGetQueueLength(), "threadC").start();
    System.out.println("getQueueLength = " + playLock.lock.getQueueLength());
    Thread.sleep(2000);

    // threadA enter doGetQueueLength()
    // getQueueLength = 1
}

@Test
public void hasQueuedThreadTest() throws InterruptedException {
    PlayLock playLock = new PlayLock();
    Thread threadA = new Thread(() -> playLock.doHasQueuedThread(), "threadA");
    threadA.start();
    Thread.sleep(10);
    Thread threadB = new Thread(() -> playLock.doHasQueuedThread(), "threadB");
    threadB.start();
    Thread.sleep(10);
    System.out.println("threadA is waiting to acquire lock? " + playLock.lock.hasQueuedThread(threadA));
    System.out.println("threadB is waiting to acquire lock? " + playLock.lock.hasQueuedThread(threadB));
    System.out.println("Has any threads waiting to acquire lock? " + playLock.lock.hasQueuedThreads());

    // threadA enter doHasQueuedThread()
    // threadA is waiting to acquire lock? false
    // threadB is waiting to acquire lock? true
    // Has any threads waiting to acquire lock? true
}

```

## `Condition`相关方法  

主要涉及如下两个方法：  

> hasWaiters(Condition condition): Queries whether any threads are waiting on the given condition associated with this lock.  
> getWaitQueueLength(Condition condition): Returns an estimate of the number of threads waiting on the given condition associated with this lock.  


``` java
public class PlayLock {
    public void doGetWaitQueueLength(){
        try {
            lock.lock();
            System.out.println(Thread.currentThread().getName() + " enter doGetWaitQueueLength()");
            condition1.await();
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }

    public void doHasWaiters(){
        try {
            lock.lock();
            System.out.println(Thread.currentThread().getName() +
                    " hasWaiter() = " + lock.hasWaiters(condition1));
            System.out.println(Thread.currentThread().getName() +
                    " getWaitQueueLength() = " + lock.getWaitQueueLength(condition1));
        } finally {
            lock.unlock();
        }
    }
}

@Test
public void hasWaitersTest() throws InterruptedException {
    PlayLock playLock = new PlayLock();
    new Thread(() -> playLock.doGetWaitQueueLength(), "threadA").start();
    new Thread(() -> playLock.doGetWaitQueueLength(), "threadB").start();
    new Thread(() -> playLock.doGetWaitQueueLength(), "threadC").start();
    new Thread(() -> playLock.doGetWaitQueueLength(), "threadD").start();
    Thread.sleep(100);
    playLock.doHasWaiters();
}

-------------  Console  --------------
// threadA enter doGetWaitQueueLength()
// threadB enter doGetWaitQueueLength()
// threadC enter doGetWaitQueueLength()
// threadD enter doGetWaitQueueLength()
// main hasWaiter() = true
// main getWaitQueueLength() = 4
```


## `Reference`  
- `《Java多线程编程核心技术》 - 4.1 使用ReentrantLock类`  
