---
layout: post
title: 并发 - [4] 锁
category: multithread
tags: [multithread]
excerpt: Concurreny Lock
---

以下内容总结自：   
  
`徐隆曦 - Java 并发编程 78 讲 - [4]各种各样的"锁"`  


## 有哪些常见的锁？  

主要分为以下七类，千万不要死记硬背，先有个印象，后面会有详细的介绍。  

- 偏向锁 / 轻量级锁 / 重量级锁  
- 公平锁 / 非公平锁
- 共享锁 / 独占锁
- 悲观锁 / 乐观锁  
- 可重入锁 / 不可重入锁  
- 自旋锁 / 非自旋锁  
- 可中断锁 / 不可中断锁  

## 悲观锁和乐观锁的本质是什么？  

先来介绍`悲观锁`：  

在获取资源之前，必须先拿到锁，以便达到`独占`的状态。  

而`乐观锁`，它并不要求在获取资源前拿到锁，也不会锁住资源。  

相反，乐观锁利用`CAS[Compare And Swap]`的思想，在不独占资源的情况下，完成了对资源的修改，过程如下：  

首先，获得原始的状态，然后，线程执行相应的计算操作，  

最后，在更新数据之前，需要检查在计算期间，原始状态是否发生了变化？  

若状态发生了变化，则放弃这次计算操作，并选择`报错`、`重试`等机制。   

若没有发生变化，则可以正常地修改数据。  

我们平常使用的`synchronized`关键字和`Lock`接口均属于`悲观锁`。  

而`原子类`则属于`乐观锁`。  

再来介绍一下它们各自的应用场景：  

- 悲观锁  

适用于`写多`，`竞争激烈`的场景，避免大量无意义的重复计算操作。  

- 乐观锁  

适用于`读多写少`，也适用于虽然读写都很多，但是`并发不激烈`的场景。  


## `synchronized`背后的`monitor`锁  

首先，需要明确`获取`和`释放 monitor锁`的时机？  

每个`Java`对象都有一个锁，这个锁也被称为`内置锁`或`monitor锁`。  

获得`monitor锁`的唯一途径就是`进入由这个锁保护的同步代码块或同步方法。`  

线程在进入被`synchronized`保护的代码块之前，会自动获取锁，  

并且无论是`正常退出`，还是通过`抛出异常退出`，都会`自动释放锁`。  

我们来看下`synchronized`代码块反编译的结果：  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/synchronized_block_decompile.png)  

可以看到有如下两个命令：  

- `monitorenter`  
- `monitorexit`    

那么它们作用分别是什么？  

`monitorenter`命令的作用是： 尝试获取`monitor锁`的控制权  

一般有三种情况：  

第一种，如果该`monitor`锁的计数为`0`，那么当前线程直接获取该`monitor锁`并将其计数设置为`1`，  

此时该线程就是这个`monitor锁`的所有者。  


第二种，如果当前线程已经拥有了这个`monitor锁`，那么它将重新进入，并累加计数。  

第三种，如果其他线程已经拥有了这个`monitor`锁，那么当前线程就会被阻塞。  


再来看下`monitorexit`命令的作用：  

它的作用是将`monitor锁`的计数`减1`，直至减为`0`，代表这个`monitor锁`已经被释放了。  


## 如何选择`synchronized`和`Lock`  

先来看下它们的相同点：  

- 都可以保证`线程安全`。  
- 都可以保证`可见性`。  
- 都有`可重入`的特点。  

再来看下它们的不同点，可以分成`7`个方面：  

第一，`用法`不同：  

`synchronized`关键字可以加在`方法`上，可以不指定锁对象`[默认为 this ]`  

也可以新建一个`同步代码块`并且`自定义 monitor 锁对象`。  

而`Lock`接口必须显式用`Lock锁对象`进行`加锁 lock() `和`解锁 unlock() `操作，  

并且一般会在`finally`块中使用`unlock()`进行`解锁`，防止`死锁`问题。  

第二，`加解锁顺序`不同：  

先来看下`synchronized`关键字  

``` java
// 获得obj1对象的monitor锁
synchronized(obj1){
    // 获得obj2对象的monitor锁
    synchronized(obj2){
        ...
    }
    // 释放obj2对象的monitor锁
}
// 释放obj1对象的monitor锁
```

再来看下`Lock`：  

``` java
// 获得lock1对象的锁
lock1.lock();
// 获得lock2对象的锁
lock2.lock();
...
// 释放lock1对象的锁
lock1.unlock();
// 释放lock2对象的锁
lock2.unlock();
```

第三点，使用时的`灵活程度`不同。  

`synchronized`不够灵活，一旦该锁被线程`A`获得了，如果此时线程`B`也想要获取该锁，  

那么它只能被阻塞，直到线程`A`运行完毕或发生异常。  

而`Lock`则比较灵活，它可以选择尝试获取锁，也可以选择中断，然后先去执行别的任务。  


第四点，  

`synchronized`的锁只能同时被一个线程拥有，而`Lock`锁没有这个限制，如`读锁`。  

第五点，`原理`不同。  

`synchronized`是`内置锁`，由`JVM`实现获取和释放锁，还分为`偏向锁`、`轻量级锁`和`重量级锁`。  

而`Lock`根据实现方式的不同，有不同的原理，例如`ReentrantLock`内部通过`AQS`来获取和释放锁。  

第六点，是否可以`设置公平策略`？  

`synchronized`不能设置，而`ReentrantLock`可以根据自己的需要来设置公平或非公平。   

第七点，性能不同。  

`JDK5`以及之前，`synchronized`的性能较差。  

`JDK6`以后，对`synchronzied`进行了很多优化，例如：`自适应自旋`、`锁消除`、`锁粗化`、`偏向锁`、`轻量级锁`等，  

所以后期的`Java`版本里`synchronized`的性能并不比`Lock`差。  

 



## `Lock`有哪些常用的方法？  

主要有如下`五`个：  

- `[void lock()]`  

当线程获取锁时，如果该锁已经被其他线程获取，则当前线程进入等待状态，否则获取该锁。  

`lock()`方法不能被中断，一旦陷入死锁，则会进入永久等待。  


- `[boolean tryLock()]`  

该方法用来尝试获取锁，如果当前锁没有被其他线程占用，则获取成功，返回`true`，  

否则返回`false`，代表获取该锁失败。  


- `[boolean tryLock(long time, TimeUnit unit)]`  

拥有`超时时间`的`tryLock()`方法。  


- `[void lockInterruptibly()]`  

除非当前线程在获取锁期间被中断，否则便会一直尝试获取锁。  

`lockInterruptibly()`是可以`响应中断`的，因此比`synchronized`关键字更加灵活。  

可以将这个方法理解成`超时时间无穷大`的`tryLock(long time, TimeUnit unit)`。  


- `[void unlock()]`  

用于`解锁`，对于`ReentrantLock`而言，执行`unlock()`时，内部会把锁的计数器`减1`，  

直至为`0`则代表当前这把锁已经被完全释放了。  



## 什么是锁的公平策略？  

先来介绍下`公平`/`非公平锁`：  

- `公平锁`  ： 按照线程的请求顺序来分配锁。  
- `非公平锁`： 不完全按照线程的请求顺序来分配锁，在一定情况下，可以允许`插队`。  

注意：这里的非公平并不是指`完全的随机`，而是仅仅`"在合适的时机"`插队。  

那么什么是`"合适的时机"`？  

假设当前线程在请求锁时，恰好前一个持有锁的线程释放了这把锁，  

那么当前申请锁的线程就可以不用等待而立即插队。  

为什么要这么设计？  

假设`线程A`持有一把锁，此时`线程B`也来请求这把锁，显然`线程B`会陷入`阻塞状态`，  

当`线程A`释放锁时，本来应该轮到`线程B`苏醒并去获取锁，  

但是此时如果忽然有一个`线程C`也来请求这把锁，  

那么根据锁的`非公平`的策略，会把这把锁给`线程C`，  

为什么？  

因为唤醒`线程B`需要较长的`时间开销`，很有可能在唤醒它之前， 

`线程C`就已经拿到了这把锁、完成了任务，并释放锁。  

相比于等待唤醒`线程B`的漫长时间，插队的行为会让`线程C`跳过陷入阻塞的过程，  

如果在锁代码中需要执行的内容不多的话，`线程C`很快就能完成任务，  

并且在`线程B`被完全唤醒之前，就把这个锁交出去，这样就是一个双赢的局面，  

对于`线程C`而言不需要等待，对于`线程B`而言，它获取锁的时间也并没有推迟， 

因为等它被唤醒的时候，`线程C`已经执行完毕并释放了锁，  

原因是`线程C`的执行速度相较于`线程B`的唤醒速度，是很快的。  

所以`Java`设计者设计了`非公平锁`，其目的是为了提升整体的运行效率。   


举个例子:  

``` java
{

    public static void main(String[] args) throws InterruptedException {
        PrintQueue pq = new PrintQueue();
        Thread[] threads = new Thread[10];

        for (int i = 0; i < 5; i++){
            threads[i] = new Thread(new Job(pq), "Thread " + i);
        }

        for (int i = 0; i < 5; i++){
            threads[i].start();
            Thread.sleep(100);
        }
    }

    static class Job implements Runnable{

        private PrintQueue queue;

        public Job(PrintQueue queue){
            this.queue = queue;
        }

        @Override
        public void run() {
            System.out.printf("%s: Going to print a job\n", Thread.currentThread().getName());
            queue.printJob();
            System.out.printf("%s: Done\n", Thread.currentThread().getName());
        }
    }

    static class PrintQueue{
//        private final Lock lock = new ReentrantLock(true); // fair
        private final Lock lock = new ReentrantLock(false); // unfair

        public void printJob(){
            lock.lock();
            try {
                long duration = (long) (Math.random() * 10000);
                System.out.printf("%s: Printing a Job during %d seconds\n",
                        Thread.currentThread().getName(), (duration / 1000));
                Thread.sleep(duration);
            } catch (InterruptedException e) {
                e.printStackTrace();
            } finally {
                lock.unlock();
            }

            lock.lock();
            try {
                long duration = (long) (Math.random() * 10000);
                System.out.printf("%s: Printing a Job during %d seconds\n",
                        Thread.currentThread().getName(), (duration / 1000));
                Thread.sleep(duration);
            } catch (InterruptedException e) {
                e.printStackTrace();
            } finally {
                lock.unlock();
            }
        }
    }

    /** Fair **/
//    Thread 0: Going to print a job
//    Thread 0: Printing a Job during 9 seconds
//    Thread 1: Going to print a job
//    Thread 2: Going to print a job
//    Thread 3: Going to print a job
//    Thread 4: Going to print a job
//    Thread 1: Printing a Job during 0 seconds
//    Thread 2: Printing a Job during 7 seconds
//    Thread 3: Printing a Job during 9 seconds
//    Thread 4: Printing a Job during 3 seconds
//    Thread 0: Printing a Job during 9 seconds
//    Thread 0: Done
//    Thread 1: Printing a Job during 3 seconds
//    Thread 1: Done
//    Thread 2: Printing a Job during 8 seconds
//    Thread 2: Done
//    Thread 3: Printing a Job during 0 seconds
//    Thread 3: Done
//    Thread 4: Printing a Job during 5 seconds
//    Thread 4: Done
//
//    Process finished with exit code 0

    /** Unfair **/
//    Thread 0: Going to print a job
//    Thread 0: Printing a Job during 4 seconds
//    Thread 1: Going to print a job
//    Thread 2: Going to print a job
//    Thread 3: Going to print a job
//    Thread 4: Going to print a job
//    Thread 0: Printing a Job during 8 seconds
//    Thread 0: Done
//    Thread 1: Printing a Job during 6 seconds
//    Thread 1: Printing a Job during 0 seconds
//    Thread 1: Done
//    Thread 2: Printing a Job during 2 seconds
//    Thread 2: Printing a Job during 8 seconds
//    Thread 2: Done
//    Thread 3: Printing a Job during 2 seconds
//    Thread 3: Printing a Job during 5 seconds
//    Thread 3: Done
//    Thread 4: Printing a Job during 9 seconds
//    Thread 4: Printing a Job during 9 seconds
//    Thread 4: Done
//
//    Process finished with exit code 0
}
```

可以看到，`公平策略`下，线程间的执行顺序是：`0123401234`  

但是在`非公平策略`下，当前线程可能发生插队现象，所以执行顺序为：`0011223344`  


不过，需要注意一个特例：`tryLock()`，它不遵守设定的公平原则。  

举个例子：  

当线程执行`tryLock()`时，一旦有线程释放了锁，  

那么该线程就可以获取锁，即使设置的是`公平锁模式`，即使在它之前有正在等待的线程，  

简单来说就是`tryLock()`可以插队。  


## 为什么要用读写锁？  

读写锁的设计是为了满足`读多写少`的场景，从而提升整体的性能。  

需要注意一点： 只有`读锁之间`才能共存！  

来看两个例子：  

``` java
public class ReadWriteLockDemo {

    private static final ReentrantReadWriteLock rReadWriteLock = new ReentrantReadWriteLock(false);

    private static void read(){
        rReadWriteLock.readLock().lock();

        try {
            System.out.println(Thread.currentThread().getName() + " 得到读锁，正在读取。");
            Thread.sleep(3000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            rReadWriteLock.readLock().unlock();
            System.out.println(Thread.currentThread().getName() + " 释放读锁。");
        }
    }

    private static void write(){
        rReadWriteLock.writeLock().lock();

        try {
            System.out.println(Thread.currentThread().getName() + " 得到写锁，正在写入。");
            Thread.sleep(3000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            rReadWriteLock.writeLock().unlock();
            System.out.println(Thread.currentThread().getName() + " 释放写锁。");
        }
    }

    public static void main(String[] args) {
        new Thread(() -> read(),  "Thread1").start();
        new Thread(() -> read(),  "Thread2").start();
        new Thread(() -> write(), "Thread3").start();
        new Thread(() -> write(), "Thread4").start();
    }
}

// ------------------------------
// Thread1 得到读锁，正在读取。
// Thread2 得到读锁，正在读取。
// Thread1 释放读锁。
// Thread4 得到写锁，正在写入。
// Thread2 释放读锁。
// Thread4 释放写锁。
// Thread3 得到写锁，正在写入。
// Thread3 释放写锁。

// Process finished with exit code 0

```

可以看到`线程1`和`线程2`都是`读锁`，所以可以同时执行，  

`线程3`和`线程4`都是`写锁`，所以`线程4`需要等到`线程3`执行完毕后，才开始运行。  

也有可能是`线程3`等待`线程4`执行完毕，因为虽然`线程3`在`线程4`之前被执行，  

但是实际的运行顺序是随机的。  


`读写不能共存`：  


``` java
public class ReadLockWaitDemo {

    private static final ReentrantReadWriteLock rReadWriteLock = new ReentrantReadWriteLock(false);

    private static void read(){
        rReadWriteLock.readLock().lock();

        try {
            System.out.println(Thread.currentThread().getName() + "得到读锁，正在读取。");
            Thread.sleep(3000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            rReadWriteLock.readLock().unlock();
            System.out.println(Thread.currentThread().getName() + "释放读锁。");
        }
    }

    private static void write(){
        rReadWriteLock.writeLock().lock();

        try {
            System.out.println(Thread.currentThread().getName() + "得到写锁，正在写入。");
            Thread.sleep(3000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            rReadWriteLock.writeLock().unlock();
            System.out.println(Thread.currentThread().getName() + "释放写锁。");
        }
    }

    public static void main(String[] args) {
        new Thread(() -> read(),  "Thread1").start();
        new Thread(() -> read(),  "Thread2").start();
        new Thread(() -> write(), "Thread3").start();
        new Thread(() -> read(),  "Thread4").start();
    }
}

// ------------------------------
// Thread1得到读锁，正在读取。
// Thread2得到读锁，正在读取。
// Thread1释放读锁。
// Thread3得到写锁，正在写入。
// Thread2释放读锁。
// Thread3释放写锁。
// Thread4得到读锁，正在读取。
// Thread4释放读锁。

// Process finished with exit code 0
```

注意，`读写不能共存`，所以`线程4`虽然是`读锁`，  

但是仍然需要等待`线程3`的`写锁`执行完毕后，才会运行。  

接着，再来解释下`锁的升降级`：  

先来看下`写锁降级为读锁`：  

``` java
public class LockDowngrade {

    Object data;
    volatile boolean cacheValid;
    final ReentrantReadWriteLock rrwl = new ReentrantReadWriteLock(false);

    void processCachedData(){
        rrwl.readLock().lock();
        if (!cacheValid){
            // 在获取写锁之前，必须释放读锁。
            rrwl.readLock().unlock();
            rrwl.writeLock().lock();
            try {
                // 防止在释放读锁，获取写锁的间隙，别的线程修改了数据。
                if (!cacheValid){
                    data = new Object();
                    cacheValid = true;
                }
                // 在不释放写锁的情况下，直接获取读锁，这就是读写锁的降级。
                rrwl.readLock().lock();
            } finally {
                // 释放写锁，但仍持有读锁。
                rrwl.writeLock().unlock();
            }
        }
        try {
            System.out.println(data);
        } finally {
            // 释放读锁。
            rrwl.readLock().unlock();
        }
    }
}
```

这么做的目的是什么？  

我的理解是，降低了锁的粒度，提升读取的性能。  

再来思考一个问题：  

- 为什么`ReentrantReadWriteLock`不支持锁的升级？  

首先，我们必须明确，`读写锁之间`不能共存。 

因此如果想将`读锁`升级为`写锁`，则需要等待所有的`读锁`都释放完毕。  

假设有`A`、`B`和`C`三个线程，它们都持有读锁。  

假设`线程A`想从`读锁`升级到`写锁`，那么它必须等到`线程B和C`都释放掉`读锁`才行，  

但是如果`线程B`也想升级成`写锁`，那么就会造成`死锁`现象。  

所以`ReentrantReadWriteLock`不支持将`读锁`升级为`写锁`。  


## 什么是自旋锁？  

`自旋锁`就是当前线程不释放`CPU`资源，而是不断循环尝试获取锁。  

来看下例子：  


``` java
public class ReentrantSpinLock {

    // 当前锁被哪个线程锁持有
    private AtomicReference<Thread> owner = new AtomicReference<>();

    // 锁重入次数
    private int count = 0;

    public void lock(){
        Thread t = Thread.currentThread();
        // 如果锁属于当前线程，则增加重入次数。
        if (t == owner.get()){
            count++;
            return;
        }
        // 否则代表有其他线程B想获得该锁，则一直自旋，直至线程B获取该锁。
        while (!owner.compareAndSet(null, t)){
            System.out.println(Thread.currentThread().getName() + " 正在自旋");
        }
    }

    public void unlock(){
        Thread t = Thread.currentThread();
        if (t == owner.get()){
            if (count > 0){
                count--;
            } else {
                owner.set(null);
            }
        }
    }

    public static void main(String[] args) {
        ReentrantSpinLock spinLock = new ReentrantSpinLock();
        Runnable r = () -> {
            System.out.println(Thread.currentThread().getName() + " 开始尝试获取自旋锁");
            spinLock.lock();
            try {
                System.out.println(Thread.currentThread().getName() + " 获取到了自旋锁");
                Thread.sleep(3000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            } finally {
                spinLock.unlock();
                System.out.println(Thread.currentThread().getName() + " 释放了自旋锁");
            }
        };

        new Thread(r, "Thread1").start();
        new Thread(r, "Thread2").start();
    }
}
```

`线程2`一直会自旋等待`线程1`执行完毕。  


那么自旋锁有什么好处？  

`自旋锁`用循环不断地尝试获取锁，让当前线程始终处于`Runnable`状态，从而节省了线程状态切换带来的开销。  

那么`自旋锁`有什么缺点？   

一旦`自旋时间过长`，则会带来很大的资源浪费。  

最后，`自旋锁`有哪些使用场景？  

它适用于`临界区比较小`的情况，避免长时间无意义的`空转`，导致资源的浪费。  



## 锁在`JDK6`版本进行了哪些优化？  

主要包含以下四个方面： 

第一，使用了`自适应的自旋锁`。  

什么是`自适应的自旋锁`？  

自旋时间不再固定，而是会根据最近自旋尝试的`成功率`、`失败率`以及当前锁拥有者的状态等多种因素决定。  

目的还是防止长时间无意义的空转，导致大量`CPU`资源的浪费。  

第二，使用了`锁消除`。  

老规矩，什么是`锁消除`？  

如果编译器能确定某个方法或变量只会在一个线程中使用，  

那么它肯定是`线程安全`的，所以我们的编译器就会做出优化，  

把相应的`同步方法`、`锁消除`，省去了加锁和解锁的操作，从而提升整体效率。  

举个例子：  

`StringBuffer`类中的实例方法`append()`  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/stringBuffer_append.png)  


第三，使用了`锁粗化`。  

什么是`锁粗化`？  

来看个例子：  

``` java
public class CoarsenLockDemo {

    public void coarsenLock(){

        synchronized (this){
            // Do something.
        }

        synchronized (this){
            // Do something.
        }

        synchronized (this){
            // Do something.
        }
    }
}
```

此时，完成可将三个`synchronized`代码块合并成一个，避免三次`加解锁`。  

但是，不建议`粗化`循环场景。  

第四，再回顾一下`偏向锁`、`轻量级锁`和`重量级锁`。  


首先，明确一点，`偏向锁`只是一个标识。  

如果一直是`线程1`来操作这个对象`o1`，那么肯定是线程安全的， 

所以仅需要标记一下即可，不需要用锁。  

但是，如果`线程2`也来访问并修改对象`o2`，此时可能是`线程1`和`线程2`交替执行，  

那么先尝试用`CAS`的方式来获取锁，所以此时升级为`轻量级锁`。  

但是，一旦竞争非常激烈，或者线程等待时间很长，那么就会造成大量资源的浪费，  

此时就会升级为`重量级锁`。  

