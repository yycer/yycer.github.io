---
layout: post
title: Java8 - 新的日期和时间
category: java-basic
tags: [spring, java]
excerpt: Java8 New Date And Time 
---

## `2020_0306更新理解`  

这次主要补充下`Date`与`LocalDate/LocalDateTime`的区别：  

首先，后者更为直观：   


``` java
@Test
void dateVsLocalDateTimeTest(){
    Date date = new Date();
    System.out.println(date);
    date.setMonth(0);
    date.setHours(0);
    date.setMinutes(0);
    date.setSeconds(0);
    System.out.println(date);

    LocalDateTime now = LocalDateTime.now();
    System.out.println(now);

    now = now.withMonth(1);
    System.out.println(now);
}

// Fri Mar 06 18:39:35 CST 2020
// Mon Jan 06 00:00:00 CST 2020
// 2020-03-06T18:39:35.964169100
// 2020-01-06T18:39:35.964169100
```

可以看到，`Date`中`Month=0`代表的是一月。  

第二点，比较重要：`LocalDateTime`是`线程安全`的。  

首先`LocalDateTime`被`final`关键字修饰，说明什么？  

它没有子类。  

再看下它的构造函数：  

```
/**
 * Constructor.
 *
 * @param date  the date part of the date-time, validated not null
 * @param time  the time part of the date-time, validated not null
 */
private LocalDateTime(LocalDate date, LocalTime time) {
    this.date = date;
    this.time = time;
}
```

说明什么？  

不能直接通过构造器生成`LocalDateTime`对象，只能通过内部的方法调用生成。  

再来看下它的两个实例成员变量：  

```java
/**
 * The date part.
 */
private final LocalDate date;
/**
 * The time part.
 */
private final LocalTime time;
```

说明什么？  

首先，`final`修饰的变量只能被赋值一次。  

再来，这两个都是实例成员变量，那么被赋值的场景有哪些？  

- 构造器  
- 声明时赋值  
- 实例初始化代码块块  

很明显，当下它只能通过构造器赋值，通过这种不可变，进而保证了线程安全。  

其他的优点，比如更丰富的方法支持、链式方法调用等，这些还需要在项目中体会体会~  



## 计算程序执行时间  

``` java
@Test
public void durationTest() throws InterruptedException {
    LocalDateTime start = LocalDateTime.now();
    Thread.sleep(1024);
    LocalDateTime end = LocalDateTime.now();
    
    Duration duration = Duration.between(start, end);
    long millis = duration.toMillis();
    System.out.println("Duration is " + millis + " ms");
    // Duration is 1024 ms
}
```

## 操纵日期  

``` java
@Test
public void manipulateDateTest(){
    // 2014-03-18
    LocalDate date = LocalDate.of(2014, 3, 18);
    // 2014-09-18
    date = date.with(ChronoField.MONTH_OF_YEAR, 9);
    // 2016-09-08
    date = date.plusYears(2).minusDays(10); 
    date.withYear(2011);
    System.out.println(date);
}
```

咋一看，`date`不就是`2011-09-08`么。  

好吧，成功被骗！  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/withYear_ignored.png)  

`date.withYear(2011)`会创建一个新的`LocalDate`实例，但是它没有将这个新值赋给任何变量，导致`date`仍然是`2016-09-08`!  

``` java
@Test
public void manipulateDateTest(){
    ...
    date = date.withYear(2011);
    System.out.println(date); // 2011-09-08
}
```


## 解析和格式化时间  

- `String -> LocalDateTime`  

将`固定格式的时间字符串`解析为`LocalDateTime`对象。  

``` java
@Test
public void dateTimeFormatterTest(){
    // Step1: String -> LocalDateTime
    // "2020-02-13 11:22:33" -> 2020-02-13T11:22:33
    String dtStr1 = "2020-02-13 11:22:33";
    LocalDateTime parsedDt1 = LocalDateTime
            .parse(dtStr1, DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss"));
    System.out.println("parsedDt1 = " + parsedDt1);

    // "20200213091122" -> 2020-02-13T09:11:22
    String dtStr2 = "20200213091122";
    LocalDateTime parsedDt2 = LocalDateTime
            .parse(dtStr2, DateTimeFormatter.ofPattern("yyyyMMddHHmmss"));
    System.out.println("parsedDt2 = " + parsedDt2);

    // parsedDt1 = 2020-02-13T11:22:33
    // parsedDt2 = 2020-02-13T09:11:22
}
```


- `LocalDateTime -> String`  

将`LocalDateTime`对象`格式化`成指定的时间字符串。  

``` java
@Test
public void dateTimeFormatterTest(){
    // Step2: LocalDateTime -> String
    // 2020-02-13T11:22:33  -> "2020-02-13 11:22:33"
    String formattedDt1 = parsedDt1.format(DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss"));
    System.out.println("formattedDt1 = " + formattedDt1);

    // 2020-02-13T09:11:22  -> "20200213091122"
    String formattedDt2 = parsedDt2.format(DateTimeFormatter.ofPattern("yyyyMMddHHmmss"));
    System.out.println("formattedDt2 = " + formattedDt2);

    // formattedDt1 = 2020-02-13 11:22:33
    // formattedDt2 = 20200213091122
}
```

慢着，我心中一直有个疑问：中间的`T`是什么东西？  

我们来看下`LocalDateTime`中`toString()`方法：  


![](https://yyc-images.oss-cn-beijing.aliyuncs.com/localDateTime_toString.png)  
原来是这样，前面`Date`，后面是`Time`，中间的`T`就代表`Time`的开始。  


## 时区  

``` java
@Test
public void zoneIdTest(){
    LocalDateTime localDateTime = LocalDateTime.now();
    System.out.println("localDateTime = " + localDateTime);
    // localDateTime = 2020-02-13T09:25:00.626

    LocalDateTime utcDateTime = LocalDateTime.now(ZoneOffset.UTC);
    System.out.println("utcDateTime = " + utcDateTime);
    // utcDateTime = 2020-02-13T01:25:00.626

    ZoneId tokyoZoneId = ZoneId.of("Asia/Tokyo");
    ZonedDateTime tokyoNowByZoneId = utcDateTime.atZone(tokyoZoneId);
    System.out.println("tokyoNowByZoneId = " + tokyoNowByZoneId);
    // tokyoNowByZoneId = 2020-02-13T01:25:00.626+09:00[Asia/Tokyo]

    LocalDateTime tokyoNowByZoneOffset = LocalDateTime.now(ZoneOffset.of("+9"));
    System.out.println("tokyoNowByZoneOffset = " + tokyoNowByZoneOffset);
    // tokyoNowByZoneOffset = 2020-02-13T10:25:00.626

    ZoneId parisZoneId = ZoneId.of("Europe/Paris");
    ZonedDateTime parisDateTime = utcDateTime.atZone(parisZoneId);
    System.out.println("parisDateTime = " + parisDateTime);
    // parisDateTime = 2020-02-13T01:25:00.626+01:00[Europe/Paris]

    ZoneId losAngelesZoneId = ZoneId.of("America/Los_Angeles");
    ZonedDateTime losAngelesDateTime = utcDateTime.atZone(losAngelesZoneId);
    System.out.println("losAngelesDateTime = " + losAngelesDateTime);
    // losAngelesDateTime = 2020-02-13T01:25:00.626-08:00[America/Los_Angeles]
}
```

## `Reference`
- [Java8的日期LocalDateTime的中间的`T`?](https://segmentfault.com/q/1010000002909777){:target="_blank"}  
- `《Java8实战》 - 12 新的日期和时间API`  