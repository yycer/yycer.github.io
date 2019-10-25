---
layout: post
title: 多线程 - 线程生命周期
category: multithread
tags: [multithread]
excerpt: Thread life cycle
---

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/multi-threads-life-cycle.png)

## `NEW`、 `RUNNABLE`、 `TERMINATED`  

> New       : Thread state for a thread which has not yet started.  
> RUNNABLE  : Thread state for a runnable thread.  
> TERMINATED: The thread has completed execution.

也就是说，当你创建一个`Thread`对象，尚未执行`start()`方法前，该线程处于`NEW`状态。  
`RUNNABLE`代表线程进入运行的状态，  
而`TERMINATED`代表线程被销毁时的状态，比如：线程执行完毕。  

下面来看两个例子：  


``` java
@Test
public void testNewState(){
    Thread thread = new Thread(() -> System.out.println("Thread is running."));
    thread.setName("Thread1");
    System.out.println("Thread name is " + thread.getName() + " , state is " + thread.getState());
    thread.start();
    System.out.println("Thread name is " + thread.getName() + " , state is " + thread.getState());
}

--------------  Console  --------------
//        Thread name is Thread1 , state is NEW
//        Thread name is Thread1 , state is RUNNABLE
//        Thread is running.
```
<br>

``` java
@Test
public void testTerminatedState() throws InterruptedException {

    Runnable run = () -> System.out.println("Thread is running.");
    Thread thread = new Thread(run);
    thread.setName("Thread1");
    thread.start();
    Thread.sleep(1000);
    System.out.println("Thread name is " + thread.getName() + " , state is " + thread.getState());
}

--------------  Console  --------------
//        Thread is running.
//        Thread name is Thread1 , state is TERMINATED
```

<br>
## `TIMED_WAITING`  

    /**
     * Thread state for a waiting thread with a specified waiting time.
     * A thread is in the timed waiting state due to calling one of
     * the following methods with a specified positive waiting time:
     * <ul>
     *   <li>{@link #sleep Thread.sleep}</li>
     *   <li>{@link Object#wait(long) Object.wait} with timeout</li>
     *   <li>{@link #join(long) Thread.join} with timeout</li>
     *   <li>{@link LockSupport#parkNanos LockSupport.parkNanos}</li>
     *   <li>{@link LockSupport#parkUntil LockSupport.parkUntil}</li>
     * </ul>
     */

当线程需要等待一段固定的时间，则其处于`TIMED_WAITING`状态。  

- 使用`Thread.sleep()`验证`TIMED_WAITING`状态。  

``` java
/**
 * Thread1 is executed first, and thread2 is in the TIMED_WAITING state.
 */
@Test
public void testTimedWaitingStateUsingSleep() throws InterruptedException {

    Thread thread1 = new Thread(() -> {
        System.out.println(Thread.currentThread().getName() + " is running.");
        try {
            Thread.sleep(100);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println(Thread.currentThread().getName() + "Thread is end.");
    });

    Thread thread2 = new Thread(() -> {
        System.out.println(Thread.currentThread().getName() + " is running.");
        try {
            Thread.sleep(500);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println(Thread.currentThread().getName() + "Thread is end.");
    });

    thread1.setName("Thread1");
    thread2.setName("Thread2");
    thread1.start();
    thread2.start();
    Thread.sleep(150);  // The purpose is to let the thread1 finish running.
    System.out.println(thread2.getName() + " state is " + thread2.getState());
    Thread.sleep(1000); // The purpose is to let the thread2 finish running.
}

-------------  Console  --------------
//        Thread2 is running.
//        Thread1 is running.
//        Thread1Thread is end.
//        Thread2 state is TIMED_WAITING
//        Thread2Thread is end.
```

- 使用`Object.wait`验证`TIMED_WAITING`状态。  

``` java
/**
 * Before thread2 notifies thread1, thread1 is in the TIMED_WAITING state.
 */
@Test
public void testTimedWaitingStateUsingWait() throws InterruptedException {
    Object o = new Object();

    Thread thread1 = new Thread(() -> {
        System.out.println(Thread.currentThread().getName() + " is running.");
        try {
            synchronized (o){
                o.wait(1000);
                System.out.println(Thread.currentThread().getName() + " is doing something.");
            }
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println(Thread.currentThread().getName() + " is end.");
    });

    Thread thread2 = new Thread(() -> {
        System.out.println(Thread.currentThread().getName() + " is running.");
        synchronized (o){
            o.notify();
        }
        System.out.println(Thread.currentThread().getName() + " is end.");
    });

    thread1.start();
    thread1.setName("Thread1");
    thread2.setName("Thread2");
    Thread.sleep(200);
    System.out.println("The state of " + thread1.getName() + " is " + thread1.getState());
    thread2.start();
}

--------------  Console  --------------
//        Thread1 is running.
//        The state of Thread1 is TIMED_WAITING
//        Thread2 is running.
//        Thread2 is end.
//        Thread1 is doing something.
//        Thread1 is end.
```

<br>
## `WAITING`  

>     /**
     * Thread state for a waiting thread.
     * A thread is in the waiting state due to calling one of the
     * following methods:
     * <ul>
     *   <li>{@link Object#wait() Object.wait} with no timeout</li>
     *   <li>{@link #join() Thread.join} with no timeout</li>
     *   <li>{@link LockSupport#park() LockSupport.park}</li>
     * </ul>
     */

<br>
`WAITING`与`TIMED_WAITING`状态类似，只不过是无限期等待，我们来看个例子：  
``` java
@Test
public void testWaitingStateUsingWait() throws InterruptedException {

    Object o = new Object();

    Thread thread1 = new Thread(() -> {
        synchronized (o){
            try {
                System.out.println(Thread.currentThread().getName() + " is running.");
                o.wait();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    });

    Thread thread2 = new Thread(() -> {
        synchronized (o){
            System.out.println(Thread.currentThread().getName() + " is running.");
            o.notify();
        }
    });

    thread1.setName("Thread1");
    thread2.setName("Thread2");
    thread1.start();
    Thread.sleep(10); // The purpose is to let the thread1 start to run.
    System.out.println("The state of " + thread1.getName() + " is " + thread1.getState());
    thread2.start();
    Thread.sleep(10); // The purpose is to let the thread2 start to run.
    System.out.println("The state of " + thread1.getName() + " is " + thread1.getState());

}

--------------  Console  --------------
//        Thread1 is running.
//        The state of Thread1 is WAITING
//        Thread2 is running.
//        The state of Thread1 is TERMINATED
```


<br>
## `BLOCKED`  


>     /**
     * Thread state for a thread blocked waiting for a monitor lock.
     * A thread in the blocked state is waiting for a monitor lock
     * to enter a synchronized block/method or
     * reenter a synchronized block/method after calling
     * {@link Object#wait() Object.wait}.
     */

<br>
注意喔，当一个线程在等待锁资源时，它才会进入`BLOCKED`状态。  
下面我用`Thread.sleep()`方法模拟这种线程状态，这也体现了该方法的特点：`不会释放该线程占用的锁资源`。



``` java
@Test
public void testBlockedStateUsingSleep() throws InterruptedException {
    Object o = new Object();

    Thread thread1 = new Thread(() -> {
        synchronized (o){
            try {
                System.out.println(Thread.currentThread().getName() + " is running.");
                Thread.sleep(500);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    });

    Thread thread2 = new Thread(() -> {
        synchronized (o){
            System.out.println(Thread.currentThread().getName() + " is running.");
        }
    });

    thread1.setName("Thread1");
    thread2.setName("Thread2");
    thread1.start();
    Thread.sleep(10); // The purpose is to let the thread1 start to run.
    thread2.start();
    Thread.sleep(10); // The purpose is to let the thread2 start to run.
    System.out.println("The state of " + thread2.getName() + " is " + thread2.getState());
}

--------------  Console  --------------
//        Thread1 is running.
//        The state of Thread2 is BLOCKED
```

<br>
## 参考
- 《Java多线程编程核心技术》 - 7.1 线程的状态