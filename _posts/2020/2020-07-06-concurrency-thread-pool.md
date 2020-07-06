---
layout: post
title: 并发 - [3] 线程池
category: multithread
tags: [multithread]
excerpt: Concurreny Thread Pool
---

以下内容总结自：   
  
`徐隆曦 - Java 并发编程 78 讲 - [3]线程池`  


## 为什么要用线程池？  

首先，反复创建线程，会导致很大的`系统开销`，  

因为每个线程的`创建`和`销毁`都需要消耗资源和时间。  

其次，过多的线程会`占用过多的资源`，还会带来过多的`上下文切换`，因此可能会导致`系统不稳定`。


## 线程池各个参数的含义是什么？  

- `corePoolSize`：核心线程数。  
- `maximumPoolSize`：最大线程数。  
- `workQueue`：阻塞队列。  
- `keepAliveTime + 时间单位`：数量大于`corePoolSize`的空闲线程的存活时间。  
- `ThreadFactory`：线程工厂，用于创建线程。  
- `handler`：任务拒绝策略  

后面会详细介绍。  


## 线程池的四种拒绝策略？  

首先，我们需要明确`线程池`什么时候会`拒绝`我们的任务？  

主要有如下`两种`场景：  

第一，线程池已经关闭。  
第二，线程池已饱和。  

饱和的含义是指：线程池中的任务队列已满，并且工作线程数量等于`maximumPoolSize`。  

再来看下`四种`拒绝策略：  

- `Abort Policy`：直接抛出`RejectedExecutionException`，并通知你任务被拒绝。
- `Discard Policy`： 直接丢弃提交的新任务，不会给你任何的通知，因此存在一定的风险。
- `Discard Oldest Policy`： 丢弃任务队列中的头结点，通常是存活时间最长的任务，也存在一定的风险。
- `Caller Runs Policy`： 谁提交，谁执行。

介绍一下`第四种`拒绝策略的好处：  

第一，新提交的任务不会被丢弃，保证了我们的业务的完整性。  

第二，由于提交任务的线程会帮忙我们处理掉了部分任务，因此可以减缓了任务提交的速度，  

同时给线程池中的工作线程一定的缓冲时间，用来执行堆积的任务。  



## 六种常见线程池  

- `FixedThreadPool`:  线程数量固定的线程池。  

特点： `corePoolSize = maximumPoolSize`。

- `CachedThreadPool`:  可缓存的线程池。  

特点： `corePoolSize = 0, maximumPoolSize = Integer.MAX_VALUE, keepAliveTime = 60s`   


- `ScheduledThreadPool`: 可定时执行的线程池。  

特点：  支持周期性执行任务。


- `SingleThreadExecutor`: 线程数量为`1`的线程池。  

特点： `corePoolSize = maximumPoolSize = 1`


- `SingleThreadScheduledExecutor`: 线程数量为`1`，且可定时执行的线程池。  

特点：  `corePoolSize = 1, maximumPoolSize = Integer.MAX_VALUE`


- `ForkJoinPool`:  `JDK 8`新增。  

下面来介绍一下`ForkJoinPool`两个特点：  

第一，从名字就可以看出来，它具有`分治`的特点。  

第二，从数据结构角度来看，该线程池中的每个工作线程都有自己的`双端队列`：`Deque`。  

同时，它还可以做到工作线程之间的协同合作，举个例子：  

假设工作线程`t1`的执行任务非常繁重，分裂成`10`个子任务，但是工作线程`t0`比较闲，  

那么工作线程`t0`则会想办法帮助工作线程`t1`执行任务，也就是所谓的`work-stealing`。  


最后，再强调`ScheduledThreadPool`定时线程器中的三个易混淆的方法：  

第一个方法，`schedule(Runnable command, long delay, TimeUnit unit);`  

这个方法的作用是： 延迟`delay`时间后，执行一次`command`任务。  

举个例子： 

``` java
public class ScheduleTest {

    public static void main(String[] args) throws InterruptedException {

        ScheduledExecutorService scheduledThreadPool = Executors
                .newScheduledThreadPool(10);

        System.out.println("Start " + LocalDateTime.now());
        Runnable task3 = () -> {
            System.out.println("task3 " + LocalDateTime.now());
        };
        scheduledThreadPool.schedule(task3, 3, TimeUnit.SECONDS);
        System.out.println("End   " + LocalDateTime.now());

        scheduledThreadPool.awaitTermination(20, TimeUnit.SECONDS);
        scheduledThreadPool.shutdown();
    }
}

// ------------------------------
// Start 2020-07-06T16:38:26.427
// End   2020-07-06T16:38:26.436
// task3 2020-07-06T16:38:29.436

// Process finished with exit code 0
```


第二个方法，`scheduleAtFixedRate(Runnable command, long initialDelay, long period, TimeUnit unit);`  

这个方法的作用是： 先延迟`initialDelay`时间，然后每隔`period`时间就执行一次。  

举个例子： 

``` java
public class ScheduleTest {

    public static void main(String[] args) throws InterruptedException {

        ScheduledExecutorService scheduledThreadPool = Executors
                .newScheduledThreadPool(10);

        System.out.println("Start " + LocalDateTime.now());
        Runnable task3 = () -> {
            System.out.println("task3 " + LocalDateTime.now());
        };
         scheduledThreadPool.scheduleAtFixedRate(task3, 3, 5, TimeUnit.SECONDS);
        System.out.println("End   " + LocalDateTime.now());

        scheduledThreadPool.awaitTermination(20, TimeUnit.SECONDS);
        scheduledThreadPool.shutdown();
    }
}

// ------------------------------
// Start 2020-07-06T16:40:43.531
// End   2020-07-06T16:40:43.538
// task3 2020-07-06T16:40:46.538
// task3 2020-07-06T16:40:51.539
// task3 2020-07-06T16:40:56.539
// task3 2020-07-06T16:41:01.539

// Process finished with exit code 0
```

那么问题来了，如果线程执行时间大于`period`，会发生什么？  

`等待线程执行完成，并可能延迟下一次调用的时间。`  

来看例子：  

``` java
public class ScheduleTest {

    public static void main(String[] args) throws InterruptedException {

        ScheduledExecutorService scheduledThreadPool = Executors
                .newScheduledThreadPool(10);

        System.out.println("Start " + LocalDateTime.now());
        Runnable task3 = () -> {
            System.out.println("task3 " + LocalDateTime.now());
            try {
                Thread.sleep(6000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        };
         scheduledThreadPool.scheduleAtFixedRate(task3, 3, 5, TimeUnit.SECONDS);
        System.out.println("End   " + LocalDateTime.now());

        scheduledThreadPool.awaitTermination(20, TimeUnit.SECONDS);
        scheduledThreadPool.shutdown();
    }
}

// ------------------------------
// Start 2020-07-06T16:43:46.808
// End   2020-07-06T16:43:46.813
// task3 2020-07-06T16:43:49.814
// task3 2020-07-06T16:43:55.814
// task3 2020-07-06T16:44:01.814

// Process finished with exit code 0
```


第三个方法：  `scheduleWithFixedDelay(Runnable command, long initialDelay, long period, TimeUnit unit);`  

假设`任务需要运行的时间`为`t`。  

注意，它与第二个方法的区别在于，它下一次执行的时间为`t + period`。  

来看个例子：   

``` java
public class ScheduleTest {

    public static void main(String[] args) throws InterruptedException {

        ScheduledExecutorService scheduledThreadPool = Executors
                .newScheduledThreadPool(10);

        System.out.println("Start " + LocalDateTime.now());
        Runnable task3 = () -> {
            System.out.println("task3 " + LocalDateTime.now());
            try {
                Thread.sleep(3000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        };
         scheduledThreadPool.scheduleWithFixedDelay(task3, 3, 5, TimeUnit.SECONDS);

        System.out.println("End   " + LocalDateTime.now());

        scheduledThreadPool.awaitTermination(20, TimeUnit.SECONDS);
        scheduledThreadPool.shutdown();
    }
}

// ------------------------------
// Start 2020-07-06T16:55:53.413
// End   2020-07-06T16:55:53.418
// task3 2020-07-06T16:55:56.419
// task3 2020-07-06T16:56:04.419
// task3 2020-07-06T16:56:12.420

// Process finished with exit code 0
```

需要考虑一种特殊的情况，如果`t > period`，则第二、三个方法的作用是一样。  




## 为什么不应该自动创建线程池？  

这里所谓的`自动`是指： 调用`Executors`类中的静态方法直接创建线程池，如： 

`Executors.newScheduledThreadPool(10);`   

要理解这个问题，必须要非常了解常用线程池的参数：  


首先，我们先来看`FixedThreadPool`：  


``` java
public static ExecutorService newFixedThreadPool(int nThreads) {
    return new ThreadPoolExecutor(nThreads, nThreads,
                                  0L, TimeUnit.MILLISECONDS,
                                  new LinkedBlockingQueue<Runnable>());
}
```

可以看到，它用的任务队列是：`LinkedBlockingQueue`  

这是一个`无界队列`，因此一旦遇到大量的任务，就可能导致`OOM [Out Of Memory]`。  

同理，`SingleThreadExecutor`用的任务队列也是`LinkedBlockingQueue`：  

``` java
public static ExecutorService newSingleThreadExecutor() {
    return new FinalizableDelegatedExecutorService
        (new ThreadPoolExecutor(1, 1,
                                0L, TimeUnit.MILLISECONDS,
                                new LinkedBlockingQueue<Runnable>()));
}
```

然后，我们再来看`CachedThreadPool`：  

``` java
public static ExecutorService newCachedThreadPool() {
    return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                  60L, TimeUnit.SECONDS,
                                  new SynchronousQueue<Runnable>());
}
```

它的问题在于，`maximumPoolSize = Integer.MAX_VALUE`，一旦遇到大量任务，也可能造成`OOM`。  

同理，来看`ScheduledThreadPool`、`SingleThreadScheduledExecutor`：  

``` java
public ScheduledThreadPoolExecutor(int corePoolSize) {
    super(corePoolSize, Integer.MAX_VALUE, 0, NANOSECONDS,
          new DelayedWorkQueue());
}
```

总结一下，这五种常见的线程池，都可能由于`无限队列`或`最大线程数`问题，导致最终系统内存不足。  



## 合适的线程数量是多少？  

还有一个类似的问题： `CPU核心数`和`线程数`的关系是什么？  

首先，我们需要区分`任务的类型`，主要分为以下两种：  

- `CPU 密集型任务`  
- `IO  密集型任务`  

`CPU`密集型任务，一般需要消耗较长的时间，比如：加解密、压缩、复杂计算等操作。  

对于这样的任务最佳的线程数为`CPU`核心数的`1 ~ 2`倍。  

`IO`密集型任务，一般等待时间较长，比如：数据库操作、文件读写等。  

可以参考如下公式：  

`线程数 = CPU核心数 * (1 + 平均等待时间) / 平均工作时间。`  


总结一下：  

- 线程的平均`工作时间`所占比例越高，则需要`更少`的线程。  
- 线程的平均`等待时间`所占比例越高，则需要`更多`的线程。  


## 如何定制自己的线程池？  

一般需要结合`实际应用场景`，考虑如下四个方面：  

- `线程数量`: `corePoolSize`、`maximumPoolSize`
- `任务队列`: `LinkedBlockingQueue、ArrayBlockingQueue、SynchronousQueue、DelayWorkQueue`
- `线程工厂`: `Google - ThreadFactoryBuilder`
- `拒绝策略`: `Abort、Discard、DiscardOldest、CallerRun`


## 如何正确关闭线程池？  

类似的问题还有：`shutdown()`与`shutdownNow()`的区别是什么？  


下面总结一下常见的五个相应方法：  


- `1、 [void] shutdown()`  

作用： 可以安全地关闭一个线程池，但不是立即`彻底关闭`。  

若线程池仍有正在运行的任务，或者任务队列中还有正在等待被执行的任务， 

那么，调用`shutdown()`方法后，线程池仍会等待它们执行完成后，才会彻底关闭。  

不过此时线程池会根据拒绝策略直接`拒绝后续新提交的任务`。  


- `2、 [boolean] isShutdown()`  

作用： 判断线程池是否已经开始了关闭操作，若为`true`则不一定表示`彻底关闭`。

- `3、 [boolean] isTerminated()`  

作用： 用于检测线程池是否`彻底关闭`！  

- `4、 [boolean] awaitTermination()`  

作用： 等待一段时间后，判断线程池是否`彻底关闭`，分为`三种`情况：  

第一种： 等待期间 (包括进入等待状态前) 线程池已经彻底关闭，则返回`true`。  

第二种： 等待时间过后，线程池仍未处于彻底关闭状态，则返回`false`。  

第三种： 如果在等待期间，当前线程被中断，则抛出`InterruptedException`异常。  

- `5、 [List<Runnable>] shutdownNow()`  

作用如下：  

首先，中断线程池中所有工作线程。   

然后，将任务队列中尚未执行的任务保存至`List<Runnable>`中返回。  


## 线程池复用线程的原理是什么？  


先来看下示意图：  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/thread_pool_process_task.png)  

源码层面解释，后续再补充。  



