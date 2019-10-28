---
layout: post
title: 多线程 - interrupt
category: multithread
tags: [multithread]
excerpt: interrupt
---

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/multi-threads-interrupt.png)

## `interrupt()`  

首先`interrupt()`是`Thread`类中的实例方法，它的作用是设置中断标记`Just to set the interrupt flag`。  


<br>
## `interrupted()` vs `isInterrupted()`  

    interrupted():
    Tests whether the current thread has been interrupted.  
    The interrupted status of the thread is cleared by this method.  
    In other words, if this method were to be called twice in succession, the
    second call would return false.


注意喔，`interrupted()`是`Thread`类中的静态方法。该方法的作用是判断当前线程是否已中断，同时它会清除该线程的中断状态。因此，当你连续调用该方法两次，第二次返回的肯定是`false`.

<br>

    isInterrupted():
    Tests whether this thread has been interrupted.  
    The interrupted status of the thread is unaffected by this method.

`isInterrupted()`是`Thread`类中的实例方法，该方法的作用是判断该线程是否已中断，但是，该方法不会影响线程的中断状态！    

最后，让我们来看个例子:  

``` java
@Test
public void testInterrupt() throws InterruptedException {
    Thread thread1 = new Thread(() -> {

        System.out.println(Thread.currentThread().getName() + " start");
        while (!Thread.currentThread().isInterrupted()){
            System.out.println(Thread.currentThread().getName() + " is running. ");
        }
        System.out.println("Get " + Thread.currentThread().getName() +
                " interrupted status is " + Thread.currentThread().isInterrupted() +
                " using Thread.currentThread().isInterrupted()");
        System.out.println("Get " + Thread.currentThread().getName() +
                " interrupted status is " + Thread.interrupted() +
                " using Thread.interrupted()");
        System.out.println("Get " + Thread.currentThread().getName() +
                " interrupted status is " + Thread.interrupted() +
                " using Thread.interrupted()");

        System.out.println(Thread.currentThread().getName() + " end");
    }, "Thread1");

    thread1.start();
    Thread.sleep(5);
    thread1.interrupt();
    Thread.sleep(5);

    System.out.println("Get " + Thread.currentThread().getName() +
            " interrupted status is " + Thread.currentThread().isInterrupted() +
            " using Thread.currentThread().isInterrupted()");
}

-------------  Console  --------------
//        Thread1 start
//        Thread1 is running.
//        Get Thread1 interrupted status is true using Thread.currentThread().isInterrupted()
//        Get Thread1 interrupted status is true using Thread.interrupted()
//        Get Thread1 interrupted status is false using Thread.interrupted()
//        Thread1 end
//        Get main interrupted status is false using Thread.currentThread().isInterrupted()
```


由于`interrupted()`会清除当前线程的中断状态，因此第二次调用`interrupted()`返回`false`。  


