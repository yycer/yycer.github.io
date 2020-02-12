---
layout: post
title: Java8 - 比较器复合
category: java-basic
tags: [spring, java]
excerpt: Java8 Comparator Compound
---

## 准备数据  

``` java
List<Apple> inventory = Arrays.asList(
        new Apple(80,"green"),
        new Apple(155, "green"),
        new Apple(120, "red"),
        new Apple(80, "red"));
```


## 正序排序  

``` java
@Test
public void complexComparatorLambdaTest(){
    List<Apple> sortedByWeight = inventory.stream()
            .sorted(Comparator.comparing(Apple::getWeight))
            .collect(Collectors.toList());
    System.out.println(sortedByWeight);

    // [Apple{color='green', weight=80}, Apple{color='red', weight=80}, Apple{color='red', weight=120}, Apple{color='green', weight=155}]
}

```


## 倒序排序  

``` java
@Test
public void complexComparatorLambdaTest(){
     List<Apple> sortedByWeightReversely = inventory.stream()
                .sorted(Comparator.comparing(Apple::getWeight).reversed())
                .collect(Collectors.toList());
    System.out.println(sortedByWeightReversely);

    // [Apple{color='green', weight=155}, Apple{color='red', weight=120}, Apple{color='green', weight=80}, Apple{color='red', weight=80}]
}
```


## 比较器链  

比如： 先根据`重量正序`，然后根据`颜色倒序`。  


``` java
@Test
public void complexComparatorLambdaTest(){
    List<Apple> sortedByWeightAndColor = inventory.stream()
                .sorted(Comparator.comparing(Apple::getWeight)
                                  .thenComparing(Comparator.comparing(Apple::getColor)
                                  .reversed()))
                .collect(Collectors.toList());
    System.out.println(sortedByWeightAndColor);

    // [Apple{color='red', weight=80}, Apple{color='green', weight=80}, Apple{color='red', weight=120}, Apple{color='green', weight=155}]
}
```


## `Reference`  
- `《Java8实战》 - 3.8.1 比较器复合`  

