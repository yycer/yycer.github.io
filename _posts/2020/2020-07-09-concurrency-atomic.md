---
layout: post
title: 并发 - [7] 原子类
category: multithread
tags: [multithread]
excerpt: Concurreny Atomic Class
---

以下内容总结自：   
  
`徐隆曦 - Java 并发编程 78 讲 - [7]原子类`   


## 什么是原子类？有什么用？  

在编程领域，`原子性`意味着： `一组操作要么全部成功，要么全部失败。`  

`java.util.concurrent.atomic`下的类就是具有`原子性`的类，  

它们可以原子性地执行`添加`、`递增`、`递减`等操作。  

比如之前多线程下的`i++`问题，就可以使用`功能相同`且`线程安全`的`getAndIncrement()`方法优雅解决。  


那么`原子类`和`锁`相比，有什么优势？  

第一，原子类的`粒度`更细  

原子变量可以把竞争范围缩小到`变量级别`，通常情况下，锁的粒度都要比原子变量的粒度大。  


第二，原子类`效率更高`    

除了竞争激烈的情况外，使用原子类的效率通常会比锁更高，  

因为原子类底层使用`CAS`操作，不会阻塞线程。  


## 有哪几种原子类？  

一共分为`六`种：  

### Atomic*  

称之为`基本类型原子类`，包括三种：`AtomicInteger`、`AtomicLong`、`AtomicBoolean`  


以`AtomicInteger`类型为例，它是对于`int`类型的封装，并提供了`原子性的访问和更新`。  


那么`AtomicInteger`类有哪些`常用方法`？  


- `[int get()]`: 获取当前值。  
- `[int getAndSet()]`: 获取当前值，并设置新的值。  
- `[int getAndIncrement()]`: 获取当前值，并自增。  
- `[int getAndDecrement()]`: 获取当前值，并自减。  
- `[int getAndAnd(int delta)]`: 获取当前值，并加上预期的值。  
- `[boolean compareAndSet(int expect, int update)]`: 如果输入的数`[expect]`等于当前值，  
则以原子的方式将值更新为输入值`[update]`。  


### Atomic*Array   

称之为`数组类型原子类`，数组里的元素，都可以保证其原子性。

分为三种:   
- `AtomicIntegerArray`: 整型数组原子类  
- `AtomicLongArray`: 长整型数组原子类  
- `AtomicReferenceArray`: 引用类型数组原子类  


### Atomic*Reference  

称之为`引用类型原子类`，以`AtomicReference`为例，

它的作用和`AtomicInteger`并没有本质区别，  

`AtomicInteger`保证了一个`整数`的原子性，而`AtomicReferece`则保证了一个`对象`的原子性。  

也分为三种：  

- `AtomicReference`
- `AtomicStampedReference`: 是对`AtomicReference`的升级，在此基础上加了`时间戳`，用于解决`CAS`的`ABA`问题。  
- `AtomicMarkableReference`:  和`AtomicReference`类似，多了一个`绑定的布尔值`，表示该对象已删除的场景。  


### Atomic*FiledUpdater  

代表原子更新器，一共有三种：  

- `AtomicIntegerFieldUpdater`: 原子更新`整型`的更新器。  
- `AtomicLongFieldUpdater`: 原子更新`长整型`的更新器。  
- `AtomicReferenceFieldUpdater`: 原子更新`引用`的更新器。  

这里需要说明一下，它们的作用是什么？  

使普通类型的变量具有原子性，举个例子：  

``` java
public class AtomicIntegerFiledUpdaterDemo implements Runnable{

    static AtomicIntegerFieldUpdater<Score> scoreUpdater = AtomicIntegerFieldUpdater
            .newUpdater(Score.class, "score");

    static Score math;
    static Score computer;


    @Override
    public void run() {
        for (int i = 0; i < 1000; i++){
            computer.score++;
            scoreUpdater.getAndIncrement(math);
        }
    }

    public static class Score{
        volatile int score;
    }

    public static void main(String[] args) throws InterruptedException {
        math     = new Score();
        computer = new Score();
        AtomicIntegerFiledUpdaterDemo task = new AtomicIntegerFiledUpdaterDemo();
        Thread t1 = new Thread(task);
        Thread t2 = new Thread(task);
        t1.start();
        t2.start();
        t1.join();
        t2.join();
        System.out.println("普通变量的结果:   " + computer.score);
        System.out.println("升级后变量的结果: " + math.score);
//        普通变量的结果:   1994
//        升级后变量的结果: 2000
    }
}
```

可以看到将`Score`类中的`score`成员实例变量升级为`原子性`后，  

`math.score = 2000`，保证了数据的正确性，做到了线程安全。   


### Adder  

表示`加法器`，有两种：`LongAdder`和`DoubleAdder`  


### Accumulator  

表示`积累器`，也有两种：`LongAccumulator`和`DoubleAccumulator`  


## 高并发下AtomicInteger的性能问题   

先来看下`AtomicInteger`，创建`1千万`个线程，分别执行`+1`操作：  

``` java
public class AtomicLongDemo {

    public static void main(String[] args) throws InterruptedException {
        AtomicLong counter = new AtomicLong(0);
        long s = System.currentTimeMillis();
        ThreadPoolExecutor service = new ThreadPoolExecutor(
                16, 1000, 10L, TimeUnit.SECONDS,
                new LinkedBlockingDeque<>(), new ThreadPoolExecutor.CallerRunsPolicy());
        for (int i = 0; i < 10000000; i++){
            service.submit(new Task(counter));
        }
        long e = System.currentTimeMillis();
        // Cost : 7840ms, corePoolSize = 16, maximumPoolSize = 10000
        System.out.println("Cost : " + (e - s) + "ms");
        service.shutdown();
    }

    static class Task implements Runnable{

        private final AtomicLong counter;

        public Task(AtomicLong counter) {
            this.counter = counter;
        }

        @Override
        public void run() {
            counter.incrementAndGet();
        }
    }
}
```

再来看下`LongAdd`：  

``` java
public class LongAdderDemo {

    public static void main(String[] args) {
        LongAdder counter = new LongAdder();
        long s = System.currentTimeMillis();
        ThreadPoolExecutor service = new ThreadPoolExecutor(
                16, 1000, 10L, TimeUnit.SECONDS,
                new LinkedBlockingDeque<>(), new ThreadPoolExecutor.CallerRunsPolicy());
        for (int i = 0; i < 10000000; i++){
            service.submit(new LongAdderDemo.Task(counter));
        }
        long e = System.currentTimeMillis();
        // Cost : 5940ms
        System.out.println("Cost : " + (e - s) + "ms");
        service.shutdown();
    }

    public static class Task implements Runnable{

        private final LongAdder counter;

        public Task(LongAdder counter) {
            this.counter = counter;
        }

        @Override
        public void run() {
            counter.increment();
        }
    }
}
```

来梳理下区别是什么：  

先来看下示意图：  

该图来源于 `徐隆曦 - Java 并发编程 78 讲`

`[第40讲：AtomicInteger 在高并发下性能不好，如何解决？为什么？]`  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/re_flush.png)  

如果竞争非常激烈，那么`AtomicLong`执行`incrementAndGet()`操作时，  

就会频繁执行`flush`和`reflush`操作，导致消耗很多的资源，  

而且`CAS`也会经常失败，所以性能不高。  

而`LongAdder`引入了`分段锁`的概念，  

当竞争不激烈时，所有线程都是通过`CAS`对同一个`Base`变量进行修改，  

当竞争激烈时，`LongAdder`将不同线程对应到不同的`Cell`上进行修改，  

降低了冲突的概率，提高了并发性。  


## 原子类与volatile分别有哪些适用场景？  

如果仅仅是`可见性`的问题，那么可以使用`volatile`关键字解决，  

如修饰`boolean`类型的`标识位`，因为`直接的赋值操作`本身就具有`原子性`。  

如果是`组合操作`的问题，那么可以使用`原子类`，  

如会被多个线程同时操作的`计数器Counter`的场景。  


## 原子类和synchronized的区别是什么？  

可以从以下四个角度进行分析：  

第一，`原理`不同  

`原子类`保证`线程安全`的原理是利用了`CAS`操作。 

而`synchronized`则是利用了`monitor锁`。  


第二，`使用范围`不同  

`原子类`的使用方法具有局限性，  

因为一个原子类仅仅是一个`原子变量`，或者是`具有原子性的对象`，不够灵活。  

而`synchronized`的使用范围要更加广泛，它既可以`修饰一个方法`，也可以`修饰一段代码块`。  


第三，`粒度`不同  

原子变量的粒度是比较小的，它可以把竞争范围缩小到`变量级别`，  

通常情况下，`synchronized`锁的粒度都要比原子变量的粒度大。  


第四，`性能`不同  

`原子类`是典型的`乐观锁`，而`synchronized`属于悲观锁。  

悲观锁的开销是固定的，这种开销并不会线性增长，  

而乐观锁竞争失败后，并且随着时间的增加，它的资源消耗会呈线性增长。  


## Java8中的Accumulator有什么用？  

它是一个更通用版本的`Adder`，提供了`自定义的函数操作`。   

举个例子：  

``` java
public class AccumulatorDemo {

    public static void main(String[] args) throws InterruptedException {

//        LongAccumulator accumulator = new LongAccumulator((x, y) -> x + y, 0);
        LongAccumulator accumulator = new LongAccumulator(Long::sum, 0);
        ExecutorService service = Executors.newFixedThreadPool(8);
        IntStream.range(1, 10).forEach(i -> service.submit(() -> accumulator.accumulate(i)));
        Thread.sleep(100);
        System.out.println(accumulator.getThenReset());
    }
}
```

这段代码的作用是从`1`加到`9`。  

解释一下，`(x, y) -> x + y, 0`  

其中的`x`代表的是当前值，初始值为`0`，而`y`代表的是当前操作你要加的值。  

举个例子： 一开始`x = 0, y = 1`，所以`x + y = 1`，  

第二次执行，此时`x = 1`，而`y = 2`，所有`x + y = 3`，以此类推。  

最后结果为`45`  


那么`Accumulator`的有哪些应用场景？  

第一，需要执行大量的计算。  

第二，计算之间无相互依赖关系。  
