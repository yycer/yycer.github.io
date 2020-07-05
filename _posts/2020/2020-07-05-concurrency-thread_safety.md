---
layout: post
title: 并发 - [2] 线程安全
category: multithread
tags: [multithread]
excerpt: Concurreny Thread Safety
---

以下内容总结自：   
  
`徐隆曦 - Java 并发编程 78 讲 - [2]究竟什么是线程安全？`  


## 三类线程安全问题  

首先，需要明确什么是`线程安全`？  

> 如果某个对象是线程安全的，那么对于使用者而言，在使用时就不需要考虑方法间的协调问题。   

下面介绍下`3`种典型的线程安全问题：  

- 运行结果错误  
- 发布和初始化导致的线程安全问题  
- 活跃性问题  

### 运行结果错误  

举个例子：  

``` java
public class WrongResult {

    private static int ans = 0;

    public static void main(String[] args) throws InterruptedException {

        Runnable r = () -> {
            for (int i = 0; i < 10000; i++) {
                ans++;
            }
        };

        Thread t1 = new Thread(r);
        t1.start();
        Thread t2 = new Thread(r);
        t2.start();
        t1.join();
        t2.join();
        System.out.println(ans);
    }
}
```

我们期望的结果是`ans = 20000`，但是往往真实的结果都小于`20000`，为什么？  

原因在于`ans++`这个操作不是`原子性`的。  

虽然只有一行代码，但是它的执行步骤一般分为三步：  

- 第一步：读取`ans`  
- 第二步：增加`ans`  
- 第三步：保存`ans`  

一旦在整个过程中发生了中断，则会导致最终数据的不准确。  

举个例子，下图来自`徐隆曦老师的课程： Java 并发编程 78 讲`  


![](https://yyc-images.oss-cn-beijing.aliyuncs.com/wrong_result.png)  


描述下整个过程：   


首先，假设此时`ans = 1`，然后`线程1`读取`ans`变量，并执行`+1`操作。  

接着，注意`线程1`还未执行第三步`保存`操作，此时，`线程2`启动了，  

同样，`线程2`执行了读取`ans`操作，那么此时它读取的`ans`的值是多少？  

还是`1`，为什么？  

因为虽然`线程1`执行了`+1`操作，但是它还没执行`保存`操作，所以从`线程2`来看，`ans`还是`1`。  

`线程2`继续执行第二步操作：`ans + 1`，  

注意，假设此时`线程1`重新获取了`CPU`时间片，执行了第三步保存操作，  

将`ans + 1`的结果`2`保存下来。  

继续，轮到`线程2`执行，它也执行`保存`操作，保存的值还是`2`。  

所以，最终结果小于`20000`。  



### 发布和初始化导致的线程安全问题  


举个例子：  

``` java
public class WrongInit {

    private static Map<Integer, String> students;

    public WrongInit(){
        new Thread(() -> {
            students = new HashMap<>();
            students.put(1, "Frankie");
            students.put(2, "Jack");
            students.put(3, "Marion");
            students.put(4, "Alina");
        }).start();
    }

    public Map<Integer, String> getStudents(){
        return students;
    }

    public static void main(String[] args) {
        WrongInit init = new WrongInit();
        System.out.println(init.getStudents().get(1));
    }
}
```

来看下结果：  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/wrong_init.png)  


为什么？  

因为当主线程执行`get(1)`操作时，`students`这个`Map`还没有构建完成。  


### 活跃性问题  

其实它包含了三种常见的问题：  


#### 死锁  

所谓死锁就是：`两个线程都在等待对方持有的资源，但是两者都不互让。`    

举个例子：  

``` java
public class MayDeadLock {

    private Object o1;
    private Object o2;

    private void thread1() throws InterruptedException {
        synchronized (o1){
            Thread.sleep(500);
            synchronized (o2){
                System.out.println("线程1成功拿到两把锁");
            }
        }
    }

    private void thread2() throws InterruptedException {
        synchronized (o2){
            Thread.sleep(500);
            synchronized (o1){
                System.out.println("线程2成功拿到两把锁");
            }
        }
    }

    public static void main(String[] args) {
        MayDeadLock mayDeadLock = new MayDeadLock();
        new Thread(() -> {
            try {
                mayDeadLock.thread1();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }).start();

        new Thread(() -> {
            try {
                mayDeadLock.thread2();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }).start();
    }
}
```

看下结果：  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/dead_lock.png)  


#### 活锁  

什么是`活锁`？  

`活锁`是指： `所在运行的程序并没有阻塞，它始终在运行，但是一直得不到期望的结果。`  


举个例子：  

假设有一个消息队列，队列里面存放着各种各样的消息，  

但某个消息由于自身的错误而导致无法被正确处理，  

但是队列的重试机制会把它重新放到队列头中进行优先处理，  

这就会导致，无论这个消息被执行多少次，都无法被正确地处理。  


#### 饥饿  

什么是饥饿？  

饥饿是指：`线程需要某些资源，但是始终得不到，尤其是 CPU 资源。`  

在`Java`中线程有`优先级`的概念，优先级从`1`到`10`，`1`最低，`10`最高。  

如果我们将某个线程的优先级设置为`1`，那么这个线程就可始终无法获得`CPU`的资源，  

从而导致长时间无法运行。  


## 线程不安全的场景？  

下面总结一下`四种`需要特别注意的场景：  

### 访问共享变量或资源  

这个上面已经介绍过了，`ans`没有得到期望的结果。  


### 依赖时序的操作    

举个例子：  

``` java
if (map.containsKey(key)){
    map.remove(obj);
}
```

假设两个线程同时访问这段代码，此时`map`中包含`key`  

但是，当`线程1`执行`remove(key)`操作后，`线程2`再执行，就可能会造成数据问题。  



### 不同数据之间存在绑定关系    

举个典型的例子： `IP`与`端口`  


### 对方没有声明自己是线程安全的    

也举个典型的例子： `ArrayList`  


## 多线程引发的性能问题  

首先，我们必须明确使用多线程编程的目的是什么？  

提升性能，比如缩短响应时间，提升吞吐量等。  

但是如果我们使用了多线程技术，反而降低了性能，这就本末倒置了。  

下面总结三种多线程可能带来的性能问题：  

### 上下文切换  
### 缓存失效  
### 线程之间协作的开销  
