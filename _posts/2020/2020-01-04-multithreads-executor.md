---
layout: post
title: 多线程 - Executor框架
category: multithread
tags: [multithread]
excerpt: Executor Framework
---

## `BlockingQueue`  

首先介绍下阻塞队列是什么？  

> A Queue that additionally supports operations
 that wait for the queue to become non-empty when retrieving an
 element, and wait for space to become available in the queue when
 storing an element.  

它其实就是一个特殊的队列，只有当队列中有值时，才能从中移除元素，当队列有空位时，才能添加元素，否则就进入等待状态。  



- 插入和移除操作的四种处理方式  

> BlockingQueue methods come in four forms, with different ways of handling operations that cannot be satisfied immediately, but may be satisfied at some point in the future:  
> one throws an exception,  
> the second returns a special value (either null or false, depending on the operation),  
> the third blocks the current thread indefinitely until the operation can succeed,  
> and the fourth blocks for only a given maximum time limit before giving up.   


`BlockingQueue`中的方法有如下四种处理方式:  

- 抛出异常  
- 阻塞  
- 返回特殊值  
- 超时 


![](https://yyc-images.oss-cn-beijing.aliyuncs.com/blocking_queue_add_remove.png)  

再来看下四种常用的阻塞队列:  


![](https://yyc-images.oss-cn-beijing.aliyuncs.com/blocking_queue_implementations_comparison.png)  

`BlockingQueue`几个注意点：  

- A BlockingQueue does not accept null elements.  
- BlockingQueue implementations are designed to be used primarily for `producer-consumer` queues.  
- BlockingQueue implementations are `thread-safe`. All queuing methods achieve their effects atomically using internal locks or other forms of concurrency control.  

<br>
## Executor框架简介  

#### 两级调度模型  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/thread_two_layer_diapatch_model.png)  

<br>
#### Executor框架的结构[三大部分]  

- 任务: `Runnable、Callable`  
- 执行任务: `Executor、ExecutorService、ThreadPoolExecutor、ScheduledThreadPoolExecutor`
- 异步计算的结果: `Future、FutureTask`  

下面分别介绍下：  

- `[I]Runnable`  

> The Runnable interface should be implemented by any class whose instances are intended to be executed by a thread.  
> The class must define a method of no arguments called run.  

首先`Runnable`是个接口，只要有实例想被线程执行，都可以实现它的`run()`方法。  

- `[I]Callable`  

> A task that returns a result and may throw an exception.  
> Implementors define a single method with no arguments called call.  
The Callable interface is similar to Runnable, in that both are designed for classes whose instances are potentially executed by another thread.  
A Runnable, however, does not return a result and cannot throw a checked exception.  

`Runnable`与`Callable`区别就是，后者有`返回值`并且可以抛出`checked exception`。  


- `[I]Executor`  

> An object that executes submitted Runnable tasks.  
> This interface provides a way of decoupling task submission from the mechanics of how each task will be run,  
> including details of thread use, scheduling, etc.  
> An Executor is normally used instead of explicitly creating threads.

`Executor`的作用就是执行提交的`Runnable`任务，解耦任务`提交`与`执行`。  


- `[I]ExecutorService`  

> An Executor that provides methods to manage termination  
> and methods that can produce a Future for tracking progress of one or more asynchronous tasks.  

`ExecutorService`的作用是控制任务的终态，如:`shutdown()、shutdownNow()`，或追踪多个任务的进度。  


- `[I]Future`  

> A Future represents the result of an asynchronous computation.  
> Methods are provided to check if the computation is complete`isDone()`,  
> to wait for its completion`get(long timeout, TimeUnit unit)`,  
> and to retrieve the result of the computation`get() `.  
 
`Future`代表了异步计算的结果，并提供如下方法：判断计算是否完成？等待计算结果，获得计算的结果等。  


## `ThreadPoolExecutor`  

`Executor`框架最核心的类就是`ThreadPoolExecutor`，它是线程池的实现类，主要有如下五个组件构成：  

- `corePoolSize`: 核心线程池的大小  
- `maximumPoolSize`: 最大线程池的大小  
- `keepAliveTime`: 空闲线程的存活时间  
- `BlockingQueue`: 用于暂存任务的阻塞队列  
- `RejectedExecutionHandler`: 饱和策略，默认为`AbortPolicy`  

下面介绍下三种主要的`ThreadPoolExecutor`:  

![](
https://yyc-images.oss-cn-beijing.aliyuncs.com/four_thread_pools.png)


- `FixedThreadPool`  

![](
https://yyc-images.oss-cn-beijing.aliyuncs.com/newFixedThreadPool.png)

`FixedThreadPool`称之为`可重用固定线程数的线程池`。  

它与`SingleThreadExecutor`均使用`无界`的阻塞队列`LinkedBlockingQueue`，这会存在隐患，如果短时间内爆发大量请求，可能会造成内存不足。  


- `SingleThreadExecutor`  

![](
https://yyc-images.oss-cn-beijing.aliyuncs.com/newSingleThreadExecutor.png)


- `CachedThreadPool`  

![](
https://yyc-images.oss-cn-beijing.aliyuncs.com/newCachedThreadPool.png)

`CachedThreadPool`的特点就是使用了`SynchronousQueue`阻塞队列。  


> A BlockingQueue in which each insert operation must wait for a corresponding remove operation by another thread, and vice versa.  
> A synchronous queue does not have any internal capacity, not even a capacity of one. 

`SynchronousQueue`内部没有任何容量，插入和移除操作必须一一准确匹配。  

最后介绍下四种饱和策略：  

> New tasks submitted in method execute(Runnable) will be rejected when the Executor has been shut down, and also when the Executor uses finite bounds for both maximum threads and work queue capacity, and is saturated.  

也就是说当`Executor`关闭或者超过最大线程数和阻塞队列容量时，会触发饱和策略：  


- `AbortPolicy[default]`: the handler throws a runtime RejectedExecutionException upon rejection.  
- `CallerRunsPolicy`: the thread that invokes execute itself runs the task. This provides a simple feedback control mechanism that will slow down the rate that new tasks are submitted.  
- `DiscardPolicy`: a task that cannot be executed is simply dropped.  
- `DiscardOldestPolicy`: if the executor is not shut down, the task at the head of the work queue is dropped, and then execution is retried (which can fail again, causing this to be repeated.)  


## `ScheduledThreadPoolExecutor`  

> A ThreadPoolExecutor that can additionally schedule commands to run after a given delay, or to execute periodically.  

`ScheduledThreadPoolExecutor`使用`DelayQueue`，主要用于延期或定时执行任务。  

- `scheduleWithFixedDelay()`，先根据`intialDelay`执行一次，然后根据`delay`执行第二次。  
- `scheduleAtFixedRate()`，先根据`initialDelay`执行一次，然后根据`period`周期性执行。  


## FutureTask  

> A cancellable asynchronous computation.  
> This class provides a base implementation of `Future`, with methods to start and cancel a computation`run()、cancel()`, query to see if the computation is complete`isDone()`, and retrieve the result of the computation`get()`.  
> The result can only be retrieved when the computation has completed; the get methods will block if the computation has not yet completed.

## Reference  
- 《Java并发编程的艺术》 - 第十章[Executor框架]