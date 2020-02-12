---
layout: post
title: Java8 - 收集器常用方法
category: java-basic
tags: [spring, java]
excerpt: Java8 Collector Methods
---

## 准备工作  

``` java
List<Apple> inventory = Arrays.asList(
        new Apple(80,  "green"),
        new Apple(155, "green"),
        new Apple(120, "red"),
        new Apple(80,  "red"));

List<Order> orders = Arrays.asList(
        new Order(UUID.randomUUID().toString(), BigDecimal.valueOf(100.30).setScale(2, BigDecimal.ROUND_HALF_UP)),
        new Order(UUID.randomUUID().toString(), BigDecimal.valueOf(500.99).setScale(2, BigDecimal.ROUND_HALF_UP)),
        new Order(UUID.randomUUID().toString(), BigDecimal.valueOf(391.88).setScale(2, BigDecimal.ROUND_HALF_UP))
        );
```

## `Count`  

计算绿苹果的数量：  

``` java
@Test
public void countTest(){
    long greenAppleCount = inventory.stream().filter(a -> "green".equals(a.getColor())).count();
    System.out.println("greenAppleCount = " + greenAppleCount);
    // greenAppleCount = 2
}
```

## `Max/Min`  

这里需要分清楚：要的是`具体数值`，还是`满足条件的对象`。  

- 获取最重苹果的重量  

这个时候可以用到`数值流`，避免`装拆箱`！  

``` java
@Test
public void maxTest(){
    int maxWeight = inventory.stream().mapToInt(Apple::getWeight).max().orElse(0);
    System.out.println("maxWeight = " + maxWeight);
    // maxWeight = 155
}
```

- 获取最重的苹果信息  

这里可以配合使用`collect`和`comparator`相关方法，也可以避免`装拆箱`！  

``` java
@Test
public void maxTest(){
    Apple maxApple = inventory.stream().max(Comparator.comparingInt(Apple::getWeight)).orElse(null);
    System.out.println("maxApple = " + maxApple);
    // maxApple = Apple{color='green', weight=155}
}
```

## `Ave/Sum`  

注意喔，`平均数和总和一定是数值`。  

``` java
@Test
public void avgAndSumTest(){
    double aveAmountDouble = orders.stream().mapToDouble(o -> o.getAmount().doubleValue()).average().orElse(0);
    double sumAmountDouble = orders.stream().mapToDouble(o -> o.getAmount().doubleValue()).sum();

    System.out.println("aveAmountDouble = " + aveAmountDouble);
    System.out.println("sumAmountDouble = " + sumAmountDouble);
    // aveAmountDouble = 331.0566666666667
    // sumAmountDouble = 993.1700000000001
}
```
可以看到，存在精度问题，解决方案如下：  

``` java
@Test
public void avgAndSumTest(){
    double aveAmountDouble = orders.stream().mapToDouble(o -> o.getAmount().doubleValue()).average().orElse(0);
    double sumAmountDouble = orders.stream().mapToDouble(o -> o.getAmount().doubleValue()).sum();

    BigDecimal aveBg = BigDecimal.valueOf(aveAmountDouble).setScale(2, BigDecimal.ROUND_HALF_UP);
    BigDecimal sumBg = BigDecimal.valueOf(sumAmountDouble).setScale(2, BigDecimal.ROUND_HALF_UP);
        
    System.out.println("aveBg = " + aveBg);
    System.out.println("sumBg = " + sumBg);
    // aveBg = 331.06
    // sumBg = 993.17
}
```

## `GroupingBy`  

准备数据：  

``` java
public static final List<Dish> menu = Arrays.asList(
    new Dish("pork",         false, 800, Dish.Type.MEAT),
    new Dish("beef",         false, 700, Dish.Type.MEAT),
    new Dish("chicken",      false, 400, Dish.Type.MEAT),
    new Dish("french fries", true,  530, Dish.Type.OTHER),
    new Dish("rice",         true,  350, Dish.Type.OTHER),
    new Dish("season fruit", true,  120, Dish.Type.OTHER),
    new Dish("pizza",        true,  550, Dish.Type.OTHER),
    new Dish("prawns",       false, 400, Dish.Type.FISH),
    new Dish("salmon",       false, 450, Dish.Type.FISH)
);
```

- 根据类型分组  


![](https://yyc-images.oss-cn-beijing.aliyuncs.com/grouping_by_type.png)  


``` java
@Test
public void groupingByTest(){
    Map<Dish.Type, List<Dish>> groupingByType = Dish.menu.stream()
            .collect(Collectors.groupingBy(Dish::getType));

    System.out.println("groupingByType = " + groupingByType);
    // groupingByType = {FISH=[prawns, salmon], MEAT=[pork, beef, chicken], OTHER=[french fries, rice, season fruit, pizza]}
}
```

- 根据卡路里分组  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/grouping_by_calories.png)  

``` java
@Test
public void groupingByTest(){
    Map<CaloricLevel, List<Dish>> groupingByCalories = Dish.menu.stream()
            .collect(Collectors.groupingBy(d -> {
                if      (d.getCalories() <= 400) return CaloricLevel.DIET;
                else if (d.getCalories() <= 700) return CaloricLevel.NORMAL;
                else                             return CaloricLevel.FAT;
    }));
    System.out.println("groupingByCalories = " + groupingByCalories);
    // groupingByCalories = {FAT=[pork], DIET=[chicken, rice, season fruit, prawns], NORMAL=[beef, french fries, pizza, salmon]}
}
```

## `Reference`  
- `《Java8实战》 - 6 用流收集数据`  

