---
layout: post
title: 多线程 - 创建线程与线程池
category: multithread
tags: [multithread]
excerpt: Create thread and thread pool
---

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/multi-threads-create-threads.png)

## 创建线程  

- 继承`Thread`类，重写`run()`方法。  

``` sql
public class ChildThread extends Thread {

    @Override
    public void run(){
        System.out.println("The running thread name is " + Thread.currentThread().getName());
    }
}

@Test
public void createThreadUsingExtend(){

    ChildThread childThread = new ChildThread();
    childThread.start(); // The running thread name is Thread-3
}
```

<br>  

- 实现`Runnable`接口，重写`run()`方法，并将该对象作为参数传入`Thread`类的构造函数。  

``` sql
public class ImpleRunnable implements Runnable{

    @Override
    public void run(){
        System.out.println("The running thread name is " + Thread.currentThread().getName());
    }
}

@Test
public void createThreadUsingImpRunnable(){

    Thread thread = new Thread(new ImpleRunnable());
    thread.start(); // The running thread name is Thread-3
}


@Test
public void createThreadUsingImpRunnableByLambda(){

    Thread thread = new Thread(() -> {
        System.out.println("The running thread name is " + Thread.currentThread().getName());
    });
    thread.start(); // The running thread name is Thread-3
}
```

<br>  

- 线程池方式，以`newSingleThreadExecutor`为例。  

``` sql
public class ImpleCallable implements Callable<String> {

    private int i;

    public ImpleCallable(int i){
        this.i = i;
    }

    @Override
    public String call(){
       String threadName = Thread.currentThread().getName();
       return "Thread name = " + threadName + " , i = " + i;
    }
}

@Test
public void newSingleThreadExecutorTest() throws ExecutionException, InterruptedException {

    ExecutorService singleThreadExecutor = Executors.newSingleThreadExecutor();
    for(int i = 0; i < 3; i++){
        Future<String> submit = singleThreadExecutor.submit(new ImpleCallable(i));
        String result = submit.get();
        System.out.println(result);
    }
    singleThreadExecutor.shutdown();
}

--------------  Console  --------------
//    Thread name = pool-1-thread-1 , i = 0
//    Thread name = pool-1-thread-1 , i = 1
//    Thread name = pool-1-thread-1 , i = 2
```



<br>
## 四种线程池  

- `newSingleThreadExecutor` 

> Creates an Executor that uses a single worker thread operating off an unbounded queue.   
> Tasks are guaranteed to execute sequentially, and no more than one task will be active at any given time. 

也就是说，一次最多只能从线程池中捞出一个线程，去执行相应的任务。一旦来了多个任务，只能根据请求时间，依次排队，等待执行。  

<br>  

- `newFixedThreadPool`  

>  Creates a thread pool that reuses a fixed number of threads operating off a shared unbounded queue.  
>   At any point, at most nThreads threads will be active processing tasks.  
>   If additional tasks are submitted when all threads are active, they will wait in the queue until a thread is available.  

创建一个固定线程数量为`n`的线程池，最多可同时执行`n`个线程，一旦任务数超过`n`，只能根据请求时间，依次等待。  

``` sql
@Test
public void newFixedThreadPoolTest() throws ExecutionException, InterruptedException {

    ExecutorService fixedThreadPool = Executors.newFixedThreadPool(2);
    for(int i = 0; i < 3; i++){
        Future<String> submit = fixedThreadPool.submit(new ImpleCallable(i));
        String result = submit.get();
        System.out.println(result);
    }
    fixedThreadPool.shutdown();
}

--------------  Console  --------------
//    Thread name = pool-1-thread-1 , i = 0
//    Thread name = pool-1-thread-2 , i = 1
//    Thread name = pool-1-thread-1 , i = 2
```


<br>  

- `newCachedThreadPool`  

> Creates a thread pool that creates new threads as needed, but will reuse previously constructed threads when they are available.  
> These pools will typically improve the performance of programs that execute many short-lived asynchronous tasks.  
> If no existing thread is available, a new thread will be created and added to the pool.  
> Threads that have not been used for sixty seconds are terminated and removed from the cache.

这种线程池会根据你的实际情况，自动选择创建还是使用原来的线程。如果，你需要执行很多耗时不长的任务，则该线程池的效率会很高。一旦，一个线程超过`60`秒没有使用，则被回收掉。  

``` sql
@Test
public void newCachedThreadPoolTest() throws ExecutionException, InterruptedException {

    ExecutorService cachedThreadPool = Executors.newCachedThreadPool();
    for(int i = 0; i < 3; i++){
        Future<String> submit = cachedThreadPool.submit(new ImpleCallable(i));
        String result = submit.get();
        System.out.println(result);
    }
    cachedThreadPool.shutdown();
}

--------------  Console  --------------
//    Thread name = pool-1-thread-1 , i = 0
//    Thread name = pool-1-thread-2 , i = 1
//    Thread name = pool-1-thread-2 , i = 2
```

<br>  

- `newScheduledThreadPool`  

>  Creates a thread pool that can schedule commands to run after a given delay, or to execute periodically.  

这种线程池的特点是在某一时刻，或周期性地执行任务。  

第一种情况，延迟2秒执行任务。  

``` sql
@Test
public void newScheduledThreadPoolTest() throws InterruptedException {

    ScheduledExecutorService scheduledThreadPool = Executors.newScheduledThreadPool(10);
    System.out.println("Thread name: " + Thread.currentThread().getName() 
        + " , Start " + LocalDateTime.now());

    Runnable task1 = () -> System.out.println("Thread name: " + Thread.currentThread().getName() 
        + " , task1 " + LocalDateTime.now());

    scheduledThreadPool.schedule(task1, 2, TimeUnit.SECONDS);
    System.out.println("Thread name: " + Thread.currentThread().getName() 
        + " , End   " + LocalDateTime.now());

    Thread.sleep(3000);
    //  scheduledThreadPool.awaitTermination(10, TimeUnit.SECONDS);
    scheduledThreadPool.shutdown();
}

--------------  Console  --------------
//        Thread name: main ,            Start 2019-10-21T16:30:40.018
//        Thread name: main ,            End   2019-10-21T16:30:40.019
//        Thread name: pool-1-thread-1 , task1 2019-10-21T16:30:42.021 + 2
```

<br>  

第二种情况，先延迟3秒执行，然后每隔5秒执行一次。  

``` sql
@Test
public void newScheduledThreadPoolTest() throws InterruptedException {
    ScheduledExecutorService scheduledThreadPool = Executors.newScheduledThreadPool(10);

    System.out.println("Start " + LocalDateTime.now());
    Runnable task2 = () -> System.out.println("task2 " + LocalDateTime.now());
    scheduledThreadPool.scheduleAtFixedRate(task2, 3, 5, TimeUnit.SECONDS);
    System.out.println("End   " + LocalDateTime.now());

//        Thread.sleep(3000);
    scheduledThreadPool.awaitTermination(20, TimeUnit.SECONDS);
    scheduledThreadPool.shutdown();
}

--------------  Console  --------------
//        Start 2019-10-21T16:32:24.445
//        End   2019-10-21T16:32:24.446
//        task2 2019-10-21T16:32:27.448 + 3
//        task2 2019-10-21T16:32:32.446 + 5
//        task2 2019-10-21T16:32:37.448 + 5
//        task2 2019-10-21T16:32:42.447 + 5
```

<br>  

第三种情况，先延迟3秒执行，然后再延迟5秒执行。  

``` sql
@Test
public void newScheduledThreadPoolTest() throws InterruptedException {
    ScheduledExecutorService scheduledThreadPool = Executors.newScheduledThreadPool(10);

    System.out.println("Start " + LocalDateTime.now());
    Runnable task3 = () -> System.out.println("task3 " + LocalDateTime.now());
    scheduledThreadPool.scheduleWithFixedDelay(task3, 3, 5, TimeUnit.SECONDS);
    System.out.println("End   " + LocalDateTime.now());

//        Thread.sleep(3000);
    scheduledThreadPool.awaitTermination(10, TimeUnit.SECONDS);
    scheduledThreadPool.shutdown();
}

--------------  Console  --------------
//        Start 2019-10-21T16:33:30.001
//        End   2019-10-21T16:33:30.003
//        task3 2019-10-21T16:33:33.003 + 3
//        task3 2019-10-21T16:33:38.004 + 5
```

<br>
## 疑问 

- `Runnable`与`Callable`接口的区别？  

第一点，`Callable`有返回值，而`Runnable`没有。  

第二点，`Callable`有`Checked Exception`，而`Runnable`没有。  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/multi-threads_runnable.png)

<br>

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/multi-threads_callable.png)


<br>  

- `Feature`接口的作用？  

> A Future represents the result of an asynchronous computation.  
> Methods are provided to check if the computation is complete, to wait for its completion, and to retrieve the result of the computation.  
> The result can only be retrieved using method get when the computation has completed, blocking if necessary until it is ready.  
> Cancellation is performed by the cancel method.  
> Additional methods are provided to determine if the task completed normally or was cancelled.  

`Future`代表一个异常执行任务的结果。  
同时它包含一些方法，比如：判断操作是否完成、等待任务完成、取消任务、或者获得操作的结果。  
你只能当任务执行完成后，才能获得操作的结果，否则该线程会阻塞`Blocking`。  

<br>
<br>
=======================================================
==================2020/01/04补充=======================
=======================================================

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/thread_pool_addition_20200104.png)

这次修改主要是结合`《Java并发编程的艺术》 - 第9章(Java中的线程池)`，增加了对线程池的理解。  

## 线程池主要处理流程  

主要涉及核心线程池`CorePoolSize`、阻塞队列`RunnableTaskQueue`、最大线程数`MaximumPoolSize`，具体流程如下图所示：  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/thread_pool_process.png)  



## 线程池的使用  

#### 创建线程池  

- `CorePoolSize`: 核心线程池的大小。  
- `MaximumPoolSize`: 线程池的最大数量，若使用无界阻塞队列，该参数失去意义。  
- `KeepAliveTime`: 线程池中工作线程空闲后，存活的时间。空闲线程包括所有大于核心线程池的线程或核心线程`allowCoreThreadTimeOut`。  
- `TimeUnit`: 线程存活时间的单位，如: `DAYS、HOURS、MINUTES、SECONDS...`  
- `RunnableTaskQueue`: 用于保存等待执行任务的阻塞队列。  
- `RejectedExecutionHandler`: 当线程池和阻塞队列都满了，实行的饱和策略，如：`AbortPolicy[直接抛出异常]、DiscardOldestPolicy[丢弃最近的一个任务]、DiscardPolicy[不处理，丢弃当前任务]...`  


#### 提交任务  

- `execute()`: 用于提交不需要返回值的任务。  
- `submit()`: 用于提交需要返回值的任务，线程池会返回一个`Future`类型的对象。  

#### 关闭线程池  

- `shutdown()`: 将线程池的状态设置为`SHUTDOWN`，然后中断所有尚未执行的任务。  
- `shutdownNow()`: 将线程池的状态设置为`STOP`，尝试中断所有正在执行和暂停的任务。  



<br>
## 参考  
- [Java Concurrency / Multithreading Basics](https://www.callicoder.com/java-concurrency-multithreading-basics/){:target="_target"}
- [Java ThreadPoolExecutor shutdownNow() Method](https://www.javatpoint.com/java-threadpoolexecutor-shutdownnow-method){:target="_target"}
- [The difference between the Runnable and Callable interfaces in Java](https://stackoverflow.com/questions/141284/the-difference-between-the-runnable-and-callable-interfaces-in-java){:target="_target"}
- 《Java并发编程的艺术》 - 第9章(Java中的线程池)  
