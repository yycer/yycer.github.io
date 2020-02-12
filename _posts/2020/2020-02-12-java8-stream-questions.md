---
layout: post
title: Java8 - 使用流
category: java-basic
tags: [spring, java]
excerpt: Java8 Using Stream
---

## 准备数据  

``` java
Trader raoul = new Trader("Raoul", "Cambridge");
Trader mario = new Trader("Mario", "Milan");
Trader alan  = new Trader("Alan" , "Cambridge");
Trader brian = new Trader("Brian", "Cambridge");

List<Transaction> transactions = Arrays.asList(
        new Transaction(brian, 2011, 300),
        new Transaction(raoul, 2012, 1000),
        new Transaction(raoul, 2011, 400),
        new Transaction(mario, 2012, 710),
        new Transaction(mario, 2012, 700),
        new Transaction(alan,  2012, 950)
);

@Setter
@Getter
public  class Trader{

    private String name;
    private String city;

    public Trader(String n, String c){
        this.name = n;
        this.city = c;
    }

    @Override
    public String toString(){
        return "Trader:" + this.name + " in " + this.city;
    }
}

@Setter
@Getter
public class Transaction{

    private Trader trader;
    private int    year;
    private int    value;

    public Transaction(Trader trader, int year, int value)
    {
        this.trader = trader;
        this.year   = year;
        this.value  = value;
    }

    @Override
    public String toString(){
        return "{" + this.trader + ", " +
          "year: " + this.year   + ", " +
          "value:" + this.value  + "}";
    }
}
```

## 流相关题目  

- 找出`2011`年发生的所有交易，并按交易额`正序排序`。  

``` java
@Test
public void streamPracticeTest(){
    List<Transaction> transactions2011 = transactions.stream()
            .filter(x -> 2011 == x.getYear())
            .sorted(Comparator.comparing(Transaction::getValue))
            .collect(Collectors.toList());
    System.out.println("transactions2011.size() " + transactions2011.size());
    // transactions2011.size() = 2
}
```

- 交易员都在哪些不同的市工作过？  

``` java
@Test
public void streamPracticeTest(){
    List<String> uniqueCities = transactions.stream()
            .map(Transaction::getTrader)
            .map(Trader::getCity)
            .distinct()
            .collect(Collectors.toList());
    System.out.println("uniqueCities = " + uniqueCities);
    // uniqueCities = [Cambridge, Milan]
}
```

`优化1： 合并map`:  

``` java
@Test
public void streamPracticeTest(){
    List<String> uniqueCities = transactions.stream()
            .map(x -> x.getTrader().getCity())
            .distinct()
            .collect(Collectors.toList());
    System.out.println("uniqueCities = " + uniqueCities);
    // uniqueCities = [Cambridge, Milan]
}
```

`优化2： 使用集合`:  

``` java
@Test
public void streamPracticeTest(){
    Set<String> uniqueCitySet = transactions.stream()
            .map(x -> x.getTrader().getCity())
            .collect(Collectors.toSet());
    System.out.println("uniqueCitySet = " + uniqueCitySet);
    // uniqueCitySet = [Milan, Cambridge]
}
```

- 查找所有来自于剑桥的交易员，并按姓名排序。  

``` java
@Test
public void streamPracticeTest(){
    List<Trader> traderFromCambridge = transactions.stream()
            .map(Transaction::getTrader)
            .distinct()
            .filter(x -> "Cambridge".equals(x.getCity()))
            .sorted(Comparator.comparing(Trader::getName))
            .collect(Collectors.toList());
    System.out.println("traderFromCambridge = " + traderFromCambridge);
    // traderFromCambridge = [Trader:Alan in Cambridge, Trader:Brian in Cambridge, Trader:Raoul in Cambridge]
}
```

`优化： 过滤 -> 映射 -> 去重 -> 排序`:  

``` java
@Test
public void streamPracticeTest(){
    List<Trader> traderFromCambridge = transactions.stream()
                .filter(tran -> "Cambridge".equals(tran.getTrader().getCity()))
                .map(Transaction::getTrader)
                .distinct()
                .sorted(Comparator.comparing(Trader::getName))
                .collect(Collectors.toList());
    System.out.println("traderFromCambridge = " + traderFromCambridge);
    // traderFromCambridge = [Trader:Alan in Cambridge, Trader:Brian in Cambridge, Trader:Raoul in Cambridge]
}
```

- 返回所有交易员的姓名字符串，按字母顺序排序。  

``` java
@Test
public void streamPracticeTest(){
    String allTraderNameString = transactions.stream()
            .map(tran -> tran.getTrader().getName())
            .distinct()
            .sorted(String::compareTo)
            .reduce("", (x, y) -> x + y);
    System.out.println("allTraderNameString = " + allTraderNameString);
    // allTraderNameString = AlanBrianMarioRaoul
}
```

`优化： Collectors.joining()`  

``` java
@Test
public void streamPracticeTest(){
   String allTraderNameString = transactions.stream()
            .map(x -> x.getTrader().getName())
            .distinct()
            .sorted()
            .collect(Collectors.joining());
    System.out.println("allTraderNameString = " + allTraderNameString);
    // allTraderNameString = AlanBrianMarioRaoul
}
```

- 有没有交易员是在米兰工作的？  

``` java
@Test
public void streamPracticeTest(){
    boolean anyTraderBasedInMilan = transactions.stream()
            .anyMatch(tran -> "Milan".equals(tran.getTrader().getCity()));
    System.out.println("anyTraderBasedInMilan = " + anyTraderBasedInMilan);
    // anyTraderBasedInMilan = true
}
```

- 打印生活在剑桥的交易员的所有交易额。  

``` java
@Test
public void streamPracticeTest(){
    transactions.stream()
                .filter(x -> "Cambridge".equals(x.getTrader().getCity()))
                .map(Transaction::getValue)
                .forEach(System.out::println);
    // 300
    // 1000
    // 400
    // 950
}
```

- 所有交易中，最高的交易额是多少？  

``` java
@Test
public void streamPracticeTest(){
    Integer highestValue = transactions.stream()
            .map(Transaction::getValue)
            .reduce(Integer::max)
            .orElse(0);
    System.out.println("highestValue = " + highestValue);
    // highestValue = 1000
}
```

上面的归纳法`reduce`求最大值，有什么缺点呢？  

答案是：`装拆箱`，注意`Integer`喔。  

`优化1： Comparator.comparingInt()`:  

``` java
@Test
public void streamPracticeTest(){
    Optional<Transaction> highestTran = transactions.stream()
            .collect(Collectors.maxBy(Comparator.comparingInt(Transaction::getValue)));
    highestTran.ifPresent(tran -> System.out.println("highestValue = " + tran.getValue()));
    // highestValue = 1000
}
```

`优化2： max(Comparator<? super T> comparator)`:  

``` java
@Test
public void streamPracticeTest(){
    Optional<Transaction> highestTran = transactions.stream().max(Comparator.comparingInt(Transaction::getValue));
    highestTran.ifPresent(tran -> System.out.println("highestValue = " + tran.getValue()));
    // highestValue = 1000
}
```

- 找到交易额最小的交易。  

``` java
@Test
public void streamPracticeTest(){
    Optional<Transaction> smallestTransOpt = transactions.stream()
            .reduce((t1, t2) -> t1.getValue() < t2.getValue() ? t1 : t2);
    Transaction smallTrans = smallestTransOpt.get();
    System.out.println(smallTrans);
    // {Trader:Brian in Cambridge, year: 2011, value:300}
}
```

`优化1： min(Comparator<? super T> comparator)`:

``` java
@Test
public void streamPracticeTest(){
    Optional<Transaction> smallestTransOpt = transactions.stream()
            .min(Comparator.comparing(Transaction::getValue));
    Transaction smallTrans = smallestTransOpt.get();
    System.out.println(smallTrans);
    // {Trader:Brian in Cambridge, year: 2011, value:300}
}
```

## `Reference`  
- `《Java8实战》 - 5.5 付诸实践`  