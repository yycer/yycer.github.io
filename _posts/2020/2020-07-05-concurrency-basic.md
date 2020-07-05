---
layout: post
title: 并发 - [1] 线程基础
category: multithread
tags: [multithread]
excerpt: Concurreny basic
---

以下内容总结自：   
  
`徐隆曦 - Java 并发编程 78 讲 - [1]线程基础升华`  

## 如何实现线程？  

先抛出两个问题：  

- 为什么说本质上只有一种实现线程的方式？  
- 实现`Runnable`接口究竟比继承`Thread`类实现线程好在哪里？  


先来看第一种： 通过实现`Runnable`接口的方式来实现线程。  

主要分为三步：  

- 第一步，将一个类实现`Runnable`接口。  
- 第二步，重写`run()`方法。  
- 第三步，将实例传到`Thread`类中就可以实现多线程。  

举个例子：  


``` java
public class RunnableThread implements Runnable{

    @Override
    public void run() {
        System.out.println("实现Runnable接口创建线程。");
    }
}

private static void createThreadUsingRunnable() {
    Thread runnableThread = new Thread(new RunnableThread());
    runnableThread.start();
}
```

这里需要注意：`Thread`类中实例方法`start()`与`run()`的区别是什么？  


- 调用`start()`才会真正创建一个新的线程。  
- 调用`run()`的作用，就像在主线程中调用一个普通方法一样。  

来看个例子：  

``` java
public class RunAndCreateDiff {

    static class MyRunnable implements Runnable{
        @Override
        public void run(){
            System.out.println(Thread.currentThread().getName());
        }
    }
    public static void main(String[] args) {
        Thread thread = new Thread(new MyRunnable());
        thread.start(); // Thread-0
        thread.run();   // main
    }
}
```

继续看第二种实现线程的方式： 继承`Thread`类。  

与第一种方法不同的是它并没有实现接口，而是继承`Thread`类，并重写了其中的`run()`方法：  

``` java
public class ExtendsThread extends Thread {

    @Override
    public void run() {
        System.out.println("继承Thread类创建线程。");
    }
}

private static void createThreadUsingThread() {
    ExtendsThread extendsThread = new ExtendsThread();
    extendsThread.start();
}
```


当然还有第三种方式： 通过`线程池`创建线程。  

但是，我们来看下`线程池`创建线程的源码：  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/thread_pool_create_threads.png)  


可以看到，本质上还有通过往`Thread`构造函数中添加实现`Runnable`接口的实例实现的。  


第四种方式： 实现`Callable`接口。  

它与`Runnable`接口的区别是什么？  

它有返回值，而`Runnable`没有。    

来看个例子：  


``` java
public class CallableTask implements Callable<Integer> {
    @Override
    public Integer call() {
        return new Random().nextInt();
    }
}

private static void createThreadUsingCallable() throws ExecutionException, InterruptedException {
    ExecutorService service = Executors.newSingleThreadExecutor();
    Future<Integer> ans = service.submit(new CallableTask());
    // 1410647188
    System.out.println(ans.get());
    service.shutdown();
}
```

我们需要明确一点，无论是实现`Runnable`还是`Callable`接口，  

它们只不过是一个`任务`而已，都需要`被执行`，所以可以将它们放到`线程池`中执行。  

而`线程池`是如何`创建线程`的，我们在上面已经讲过了。  


第五种： 通过定时器`Timer`。  

本质上还有通过继承`Thread`类创建新的线程：  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/timer_create_threads.png)  


所以，我们可以得到一个初步的结论：  

实现一个新的线程有如下两种方式：  

- 实现`Runnable`接口。  
- 继承`Thread`类。  

继续深入：  

实现`Runnable`接口，只是确定了我们需要执行什么任务。  

但是真正执行，我们还是依赖于`Thread`类。  

将实现`Runnable`接口的实例，作为形参传入`Thread`构造函数中，然后调用`start()`    

所以，我们可以说：`只有一种方式创建新的线程，那就是构造Thread类`。  

只不过实现`运行内容`的方式有所不同。  


再来讲下第二个问题：  

- 实现`Runnable`接口究竟比继承`Thread`类实现线程好在哪里？  

首先，我们从系统设计的角度出发，要尽可能做到`解耦`，  

因此，我们希望它们职责分别，`Runnable`负责线程执行的内容，而`Thread`负责线程的创建、启动、设置属性等操作。  

然后，通过`Thread`类不断地创建新的线程肯定会消耗大量资源，那么我们完全可以使用`线程池`与`Runnable`替代。  

最后，一旦通过继承`Thread`的方式创建线程，就等于牺牲了代码的`可扩展性`，因为`Java`仅支持单继承。  


## 如何正确停止线程？  

首先，我们一定要避免`强制停止`，如：`stop()`方法。  

为什么？  

因为如果`强制停止`，就会造成一些安全性的问题，同时很容易导致数据出现问题。  

最正确的停止线程的方式是使用`interrupt()`，但`interrupt()`仅仅起到通知线程停止的作用。  


对于被停止的线程而言，它拥有完全的自主权，  

它既可以选择立即停止，也可以选择一段时间后停止，也可以选择压根不停止。   


那么，如何正确地使用`interrupt()`停止线程？  


``` java
while (!Thread.currentThread().isInterrupted() && More_work_to_do) {
    // Do more work
}
```

当你执行当前线程时，需要检查该线程的`中断标记位`，如果标记位为`true`，说明什么？  

有程序想要中断该线程。  


## 线程生命周期  

先来看下示意图：  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/thread_state.png)  

注意三点：  


第一，当前线程执行`notify / notifyAll()`时，线程状态会由`Timed Waiting / Waiting`变成`Blocked`，  

因为调用这两个方法前，必须获得相应对的`monitor`锁，  

如果获得该锁，则变成`Runnable`状态，否则维持`Blocked`状态。  

第二，线程的状态必须按照箭头的方向走。  

比如线程从`New`状态不能直接进入`Blocked`状态，必须经历`Runnable`状态。  

第三，线程的`生命周期不可逆`。  

一旦进入`Runnable`状态，则不能返回`New`状态，一旦终止，则不可能再有任何状态的变化。  

所以一个线程只有一次`New`和`Terminated`状态，只有处于中间状态时，才能相互转换。  



## `wait()/notify()`  


首先，第一个问题：  

- 为什么`wait()`必须在`synchronized`保护的代码中使用？  

很简单，举个反例：  


``` java
class BlockingQueue {

    Queue<String> buffer = new LinkedList<String> ();

    public void produce(String data) {
        buffer.add(data);
        notify();
    }

    public String consume() throws InterruptedException {
        while (buffer.isEmpty()) {
            wait();
        }
        return buffer.remove();
    }
}
```

假设当前队列为空，此时`消费者线程`尝试获取产品：  

第一步，因为`buffer`为空，所以`消费者线程`尝试执行`wait()`方法。  

慢着，执行`wait()`方法之前，出现了意外情况，该`消费者线程`被调度器暂停了。  

第二步，此时`生产者线程`来工作，它往队列里面塞了一个产品，然后调用`notify()`方法。  

注意，此时并没有任何线程处于等待状态，因为之前`消费者线程`还没有执行到`wait()`方法。  

第三步，假设刚才被调度器暂停的`消费者线程`继续回来执行，则调用`wait()`， 并进入了等待状态。  

出现这种情况的根本原因在于： 判断队列是否为空、执行`wait()`方法不是一个`原子操作`！   

一旦它在中间被打断，则就不是线程安全的。  



第二个问题：  

- 为什么 `wait/notify/notifyAll()` 被定义在`Object`类中，而`sleep()`定义在`Thread`类中？

首先，每个对象都有一个`monitor`监视器的锁，它存在对象头中，这个锁是对象级别的，而不是线程级别的。  

而`wait/notify/notifyAll()`也都是锁级别的操作，它们的锁属于对象，  

所以将方法定义在`Object`类中，因为`Object`是所有对象的父类。  


第二点，如果把`wait/notify/notifyAll()`方法定义在`Thread`类中，会带来很大的局限性。  

因为一个线程可能持有多把锁，以便实现相互配合的复杂逻辑，  

如果`wait()`方法定义在`Thread`类中，如何让一个线程持有多把锁呢？  

第三个问题：  

- `wait/notify()`方法与`sleep()`方法的异同？  

首先，来说相同点：  

- 它们都会`阻塞`当前线程。  
- 它们都可以响应`interrupt`中断。  


再来梳理不同点：  

- `wait()`必须在`synchronized`保护的代码中使用，而`sleep()`没有这个要求。  
- 在同步代码中执行`wait()`方法会主动释放`monitor`锁，而`sleep()`不会。  
- `wait()`没有设置超时时间，意味着永久等待，除非遇到中断或被显式唤醒，而`sleep()`方法要求定义一个时间。  
- `wait/notify()`是`Object`类中的方法，而`sleep()`是`Thread`类的方法。  


## 生产消费者模式  

下面梳理三种`生产消费者`模式： 

- `BlockingQueue`  
- `Condition`  
- `wait/notify()`  


先来看第一种：  

- 如何用`BlockingQueue`实现生产者消费者模式  


``` java
public static void main(String[] args) {
    BlockingQueue<Object> queue = new ArrayBlockingQueue<>(10);

    Runnable producer = () -> {
        while (true){
            try {
                queue.put(new Object());
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    };

    new Thread(producer).start();
    new Thread(producer).start();

    Runnable consumer = () -> {
        while (true){
            try {
                queue.take();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    };
    new Thread(consumer).start();
    new Thread(consumer).start();
}
```

再来看第二种：  

- 如何用`Condition`实现生产者消费者模式  


``` java
{
    
    private ArrayDeque<Object> queue;
    private int max = 16;
    private ReentrantLock lock = new ReentrantLock();
    private Condition notEmpty = lock.newCondition();
    private Condition notFull  = lock.newCondition();
    
    public MyBlockingQueueForCondition(int size){
        this.max = size;
        queue = new ArrayDeque<>();
    }
    
    public void put(Object o){
        lock.lock();
        try {
            while (queue.size() == max){
                notFull.await();
            }
            queue.offer(o);
            notEmpty.signalAll();
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }
    
    public Object take(){
        lock.lock();
        try {
            while (queue.size() == 0){
                notEmpty.await();
            }
            Object item = queue.poll();
            return item;
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
        return null;
    }
}
```

第三种：  

- 如何用`wait/notify()`实现生产者消费者模式  



``` java
public class MyBlockingQueue {

    private int max;
    private ArrayDeque<Object> queue;

    public MyBlockingQueue(int size){
        this.max = size;
        queue = new ArrayDeque<>();
    }

    public synchronized void put() throws InterruptedException {
        while (queue.size() == max){
            wait();
        }
        queue.offer(new Object());
        notifyAll();
    }

    public synchronized void take() throws InterruptedException {
        while (queue.size() == 0){
            wait();
        }
        System.out.println(Thread.currentThread().getName() + " " + queue.poll());
        notifyAll();
    }
}

public class WaitStyle {

    public static void main(String[] args) {

        MyBlockingQueue queue = new MyBlockingQueue(10);
        Producer producer = new Producer(queue);
        Consumer consumer = new Consumer(queue);
        new Thread(producer).start();
        new Thread(consumer).start();

    }

    static class Producer implements Runnable{

        private MyBlockingQueue queue;

        public Producer(MyBlockingQueue queue){
            this.queue = queue;
        }

        @Override
        public void run() {
            for (int i = 0; i < 100; i++){
                try {
                    queue.put();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }
    }

    static class Consumer implements Runnable{

        private MyBlockingQueue queue;

        public Consumer(MyBlockingQueue queue){
            this.queue = queue;
        }

        @Override
        public void run() {
            for (int i = 0; i < 100; i++){
                try {
                    queue.take();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}
```







