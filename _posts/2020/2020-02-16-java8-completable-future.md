---
layout: post
title: Java8 - CompletableFuture
category: java-basic
tags: [spring, java]
excerpt: Java8 CompletableFuture
---

本文主要整理下`CompletableFuture`的基本用法，以及与`Future`和`ParallelStream`相比下的优点。  


## `Future`  

先说下`Future`的优点是什么？  

实现了异步任务，快速返回一个运算结果的引用，防止程序因调用耗时任务而阻塞。  

看到个很形象的例子，当你拿着脏衣服去干洗店时，老板会给你一张票，叫你什么时候来取，此时你就可以做自己的事情了。  


再举个具体的例子，假设你想通过产品名称从各个平台查询其最低价格。  

模拟查询产品价格时的1秒延迟： 

``` java
public class Shop {
    public String getPrice(String productName){
        delay();
        return getRandomPrice(productName);
    }

    public static void delay() {
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    private String getRandomPrice(String productName) {
        BigDecimal priceBd = BigDecimal
                .valueOf(random.nextDouble() * productName.charAt(0) + productName.charAt(1))
                .setScale(2, BigDecimal.ROUND_UP);
        return String.valueOf(priceBd);
    }
}
```

具体实现：  

``` java
public class Shop {
    public Future<String> getPriceUsingFuture(String productName){
        ExecutorService cachedThreadPool = Executors.newCachedThreadPool();
        Future<String> future = cachedThreadPool
                .submit(() -> getPrice(productName));
        return future;
    }
}

@Test
public void getPriceAsyncTest(){
    LocalDateTime step1 = LocalDateTime.now();
    Shop jd = new Shop("JD");
    Future<String> macFuturePrice = jd.getPriceUsingFuture("Mac");
    LocalDateTime step2 = LocalDateTime.now();
    System.out.println("Invocation returned after " + Duration.between(step1, step2).toMillis() + " ms.");

    try {
        String price = macFuturePrice.get();
        // String price = macFuturePrice.get(2, TimeUnit.SECONDS);
        System.out.println("Price is " + price);
    } catch (InterruptedException | ExecutionException e) {
        e.printStackTrace();
    }
    LocalDateTime step3 = LocalDateTime.now();
    System.out.println("Price returned after " + Duration.between(step1, step3).toMillis() + " ms.");
}

-------------  Console  --------------
// Invocation returned after 1 ms.
// Price is 135.87
// Price returned after 1004 ms.
```

可以看到调用`getPriceUsingFuture()`方法后，立即返回了`Future<String>`，整个过程耗时小于`1ms`。  

既然`Future`可以做到异步调用任务了，为什么还要`CompletableFuture`？  

我的理解是这样的，`CompletableFuture`的功能更强大，主要包含以下几个方面：  

- `可设置专属执行器`  
- `流水化操作多个异步任务`  
- `及时响应异步任务的能力`  


让我们看下`CompletableFuture`使用方式：  

``` java
public class Shop {
    public Future<String> getPriceAsync(String productName){
        CompletableFuture<String> futurePrice = new CompletableFuture<>();
        new Thread(() -> {
            try {
                String price = getPrice(productName);
                futurePrice.complete(price);
            } catch (Exception ex){
                futurePrice.completeExceptionally(ex);
            }
        }).start();
        return futurePrice;
    }
}
```

有更精简的写法么？  

使用`supplyAsync()`  


``` java
public class Shop {
    public Future<String> getPriceAsync(String productName){
        return CompletableFuture.supplyAsync(() -> getPrice(productName));
    }
}
```


## `ParallelStream`  

让我们同时处理多个异步任务：根据产品名称从不同的平台获取价格。  

首先来看下同步执行多个查询任务：  

``` java
public final List<Shop> shops = buildShops(4);

public List<Shop> buildShops(int num){
    return IntStream.range(0, num)
                .mapToObj(i -> new Shop(String.valueOf(i)))
                .collect(Collectors.toList());
}

public List<String> getAllPrice(){
    return shops.stream()
            .map(s -> String.format("%s:%s", productName, s.getPrice(productName)))
            .collect(Collectors.toList());
}

@Test
public void getAllPriceTest(){
    LocalDateTime start = LocalDateTime.now();
    getAllPrice(); // Using 4003 ms
    LocalDateTime end = LocalDateTime.now();
    System.out.println("Using " + Duration.between(start, end).toMillis() + " ms");
}
```

可以看到，从四个平台查询产品价格，总耗时`4003`毫秒。  

有什么优化方案么？  

之前貌似学过`并行流`，试试？  


``` java
/**
 * getAllPriceUsingParallelStream
 */
public List<String> getAllPriceUsingPS(){
    return shops.parallelStream()
            .map(s -> String.format("%s:%s", productName, s.getPrice(productName)))
            .collect(Collectors.toList());
}

@Test
public void getAllPriceTest(){
    LocalDateTime start = LocalDateTime.now();
    getAllPriceUsingPS(); // Using 1004 ms
    LocalDateTime end = LocalDateTime.now();
    System.out.println("Using " + Duration.between(start, end).toMillis() + " ms");
}
```

哇，效果很好！同样查询了四家商品的价格，却只花了`1004`毫秒！  


让我们再试下`CompletableFuture`：  


``` java
/**
 * getAllPriceUsingCompletableFuture
 */
public List<String> getAllPriceUsingCF(){
    List<CompletableFuture<String>> futurePriceList = shops.stream()
            .map(s -> CompletableFuture.supplyAsync(() -> 
                String.format("%s:%s", productName, s.getPrice(productName))))
            .collect(Collectors.toList());

    List<String> priceList = futurePriceList.stream()
            .map(CompletableFuture::join)
            .collect(Collectors.toList());

    return priceList;
}

@Test
public void getAllPriceTest(){
    LocalDateTime start = LocalDateTime.now();
    getAllPriceUsingCF(); // Using 1004 ms
    LocalDateTime end = LocalDateTime.now();
    System.out.println("Using " + Duration.between(start, end).toMillis() + " ms");
}
```

费了那么大劲，效果和并行流一样嘛。  

不不不，它们内部用的都是一样的通用线程池，线程数都是固定的，  
取决于`Runtime.getRuntime().availableProcessors()`的返回值，  
但是`CompletableFuture`可以`设置自己的执行器`！  

我电脑有效处理器的数量是`12`，让我们查询`13`个平台来看看区别：  

``` java
public final List<Shop> shops = buildShops(13);

public final Executor executor = Executors.newFixedThreadPool(
        Math.min(shops.size(), 100),
        r -> {
            Thread thread = new Thread(r);
            thread.setDaemon(true);
            return thread;
        });

@Test
public void getAllPriceTest(){
    LocalDateTime start = LocalDateTime.now();
    getAllPriceUsingPS(); // Using 2005 ms
    getAllPriceUsingCF(); // Using 1003 ms
    LocalDateTime end = LocalDateTime.now();
    System.out.println("Using " + Duration.between(start, end).toMillis() + " ms");
}
```

看到差别了吧！  


## `CompletableFuture`  

主要介绍以下两点：  

- `流水化操作多个异步任务`  
- `及时响应异步任务的能力`  


#### `流水化操作多个异步任务`  

假设我们现在需要连续进行两步操作：  

第一，通过产品名称从不同平台获得价格。  

第二，折扣平台收到价格、计算折扣并返回最终价格。  


先来看下阻塞的版本：  


``` java
@Getter
public class Quote {

    private String price;

    public Quote(String price){
        this.price = price;
    }

    public static Quote buildQuote(String price){
        return new Quote(price);
    }
}

public class Discount {

    public static String applyDiscount(Quote quote){
        Shop.delay();
        BigDecimal priceBd = BigDecimal
                .valueOf(Double.parseDouble(quote.getPrice()) * 0.8)
                .setScale(2, BigDecimal.ROUND_UP);
        return String.valueOf(priceBd);
    }
}

public List<String> getMultiInvocation() {
    return shops.stream()
            .map(shop -> shop.getPrice(productName))
            .map(Quote::buildQuote)
            .map(Discount::applyDiscount)
            .collect(Collectors.toList());
}

@Test
public void multiInvocationTest(){
    LocalDateTime start = LocalDateTime.now();
    getMultiInvocation(); // Using 8004 ms
    LocalDateTime end = LocalDateTime.now();
    System.out.println("Using " + Duration.between(start, end).toMillis() + " ms");
}
```

查询四家平台的价格并计算折扣，花了`8 = (1 + 1) * 4秒`。  

再来看下使用`CompletableFuture`的情况：  

``` java
public List<String> getMultiInvocationUsingCF() {
    List<CompletableFuture<String>> futurePriceList = shops.stream()
            .map(shop -> CompletableFuture.supplyAsync(() ->
                    shop.getPrice(productName), executor))
            .map(future -> future.thenApply(Quote::buildQuote))
            .map(future -> future.thenCompose(quote ->
                    CompletableFuture.supplyAsync(() ->
                            Discount.applyDiscount(quote), executor)))
            .collect(Collectors.toList());

    return futurePriceList.stream()
            .map(CompletableFuture::join)
            .collect(Collectors.toList());
}

@Test
public void multiInvocationTest(){
    LocalDateTime start = LocalDateTime.now();
    getMultiInvocationUsingCF(); // Using 2009 ms
    LocalDateTime end = LocalDateTime.now();
    System.out.println("Using " + Duration.between(start, end).toMillis() + " ms");
}

```

哇，同样是查询四家平台并计算折扣，`CompletableFuture`只花了`2秒`。  

而且，随着查询平台数量增加，`CompletableFuture`的优势越明显。  


#### `及时响应异步任务的能力`  

再来说下`CompletableFuture`的响应能力。  

举个例子：还是根据产品名称查询四家平台的价格，并计算其折扣，不同平台的计算速度快慢不一，只要有任何一家给出最终价格，程序就结束，这应该怎么实现？  


`CompletableFuture.anyOf()`满足你的需求。  


模拟`0.5秒`至`1.5秒`的延迟：  

``` java
public static void randomDelay(){
    int delay = 500 + Shop.random.nextInt(1000);
    try {
        Thread.sleep(delay);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
}

@Test
public void multiInvocationTest(){
    LocalDateTime start = LocalDateTime.now();
    responseTest();
    LocalDateTime end = LocalDateTime.now();
    System.out.println("Using " + Duration.between(start, end).toMillis() + " ms");
}

public void responseTest(){
    LocalDateTime start = LocalDateTime.now();
    CompletableFuture[] completableFutures = shops.stream()
            .map(shop -> CompletableFuture.supplyAsync(() ->
                    shop.getPrice(productName), executor))
            .map(fs -> fs.thenApply(Quote::buildQuote))
            .map(fq -> fq.thenCompose(quote ->
                    CompletableFuture.supplyAsync(() ->
                            Discount.applyDiscount(quote), executor)))
            .map(fs -> fs.thenAccept(s ->
                    System.out.println(s + " done in " +
                            Duration.between(start, LocalDateTime.now()).toMillis())))
            .toArray(CompletableFuture[]::new);
    // CompletableFuture.allOf(completableFutures).join();
    CompletableFuture.anyOf(completableFutures).join();
}

-------------  Console  --------------
// 1. CompletableFuture.anyOf(completableFutures).join();
// 138.49 done in 2009
// Using 2012 ms

// 2. CompletableFuture.anyOf(completableFutures).join();
// 110.65 done in 1439
// 80.20  done in 1539
// 132.33 done in 1790
// 114.11 done in 2734
// Using 2734 ms
```


## `Reference`  
- `《Java8实战》 - 11 CompletableFuture: 组合式异步变成`  

