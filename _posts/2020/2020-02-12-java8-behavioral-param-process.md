---
layout: post
title: Java8 - 行为参数化历程
category: java-basic
tags: [spring, java]
excerpt: Java8 Behavioral Param Process
---

本文梳理下由`属性参数化`到`行为参数化`的过程：  

## 准备数据  

``` java
List<Apple> inventory = Arrays.asList(
        new Apple(80,"green"),
        new Apple(155, "green"),
        new Apple(120, "red"));
```

## Step1：筛选绿苹果  

``` java
public void test(){
    List<Apple> greenApples = filterGreenApple(inventory);
    System.out.println("greenApples: " + greenApples);

    // greenApples: [Apple{color='green', weight=80}, Apple{color='green', weight=155}]
}

private List<Apple> filterGreenApple(List<Apple> inventory) {
    ArrayList<Apple> result = new ArrayList<>();
    for (Apple a: inventory){
        if ("green".equals(a.getColor())){
            result.add(a);
        }
    }
    return result;
}
```


## Step2：把颜色作为参数  

``` java
public void test(){
    List<Apple> redApples = filterAppleByColor(inventory, "red");
    List<Apple> greenApples = filterAppleByColor(inventory, "green");
    System.out.println("redApples: "  + redApples);
    System.out.println("greenApples: " + greenApples);

    // redApples  : [Apple{color='red', weight=120}]
    // greenApples: [Apple{color='green', weight=80}, Apple{color='green', weight=155}]
}

private List<Apple> filterAppleByColor(List<Apple> inventory, String color) {
    ArrayList<Apple> result = new ArrayList<>();
    for (Apple a: inventory){
        if (color.equals(a.getColor())){
            result.add(a);
        }
    }
    return result;
}
```


## Step3：筛选多个属性  

``` java
public void test(){
    List<Apple> heavyApples = filterAppleByWeight(inventory, 150);
    System.out.println("heavyApples: " + heavyApples);

    // heavyApples: [Apple{color='green', weight=155}]
}

private List<Apple> filterAppleByWeight(List<Apple> inventory, int weight) {
    ArrayList<Apple> result = new ArrayList<>();
    for (Apple a: inventory){
        if (a.getWeight() > weight){
            result.add(a);
        }
    }
    return result;
}

public void test(){
    List<Apple> greenApples = filterApple(inventory, "green", 0, true);
    List<Apple> heavyApples = filterApple(inventory, "", 150, false);
    System.out.println("greenApples: " + greenApples);
    System.out.println("heavyApples: " + heavyApples);

    // greenApples: [Apple{color='green', weight=80}, Apple{color='green', weight=155}]
    // heavyApples: [Apple{color='green', weight=155}]
}

/**
 * 根据多个条件筛选数据源
 * @param inventory：数据源
 * @param color：颜色属性
 * @param weight：重量属性
 * @param flag：true[颜色]、false[重量]
 */
private List<Apple> filterApple(List<Apple> inventory, String color, int weight, boolean flag) {
    ArrayList<Apple> result = new ArrayList<>();
    for (Apple a: inventory){
        if ((flag  && color.equals(a.getColor()) ||
            (!flag && a.getWeight() > weight))){
            result.add(a);
        }
    }
    return result;
}
```


## Step4：根据抽象条件筛选  

从这就要开始引入`行为参数化`概念了，也就是说将一个行为作为参数，传入一个方法中。  

回忆下我们之前做的事情，判断`苹果的颜色`是否为`绿色的`？`苹果的重量`是否超过`150克`？  

将其抽象成一个方法，传入的参数是`苹果`，返回一个`boolean值`，这种抽象方式称之为谓词`Predicate`。  


我们先来创建一个用于苹果的谓词：  

``` java
public interface ApplePredicate {
    /**
     * @param a 元素值
     * @return 是否满足谓词条件
     */
    boolean test(Apple a);
}
```

然后分别实现`苹果的颜色`为`绿色的`、`苹果的重量大于150克`的实现类：  

``` java
public class GreenApplePredicate implements ApplePredicate {
    @Override
    public boolean test(Apple a) {
        return "green".equals(a.getColor());
    }
}

public class HeavyApplePredicate implements ApplePredicate {
    @Override
    public boolean test(Apple a) {
        return a.getWeight() > 150;
    }
}
```

最后将其应用于`filterApple()`方法中：  


``` java
public void test(){
    List<Apple> greenApples = filterApple(inventory, new GreenApplePredicate());
    List<Apple> heavyApples = filterApple(inventory, new HeavyApplePredicate());
    System.out.println("greenApples: " + greenApples);
    System.out.println("heavyApples: " + heavyApples);
    // greenApples: [Apple{color='green', weight=80}, Apple{color='green', weight=155}]
    // heavyApples: [Apple{color='green', weight=155}]
}

/**
 * 根据谓词筛选数据源，做到行为参数化。
 * @param inventory：数据源
 * @param predicate：谓词
 */
private List<Apple> filterApple(List<Apple> inventory, ApplePredicate predicate) {
    ArrayList<Apple> result = new ArrayList<>();
    for (Apple a: inventory){
       if (predicate.test(a)){
            result.add(a);
        }
    }
    return result;
}
```

下面均使用该`List<Apple> filterApple(List<Apple> inventory, ApplePredicate predicate)`方法。  



## Step5：匿名类  

每次都要创建一个`只用一次`的谓词实现类，是不是很繁琐？  

没关系，`匿名类`可以解决这个问题！  


``` java
public void test(){
    List<Apple> greenApples = filterApple(inventory, new ApplePredicate() {
        @Override
        public boolean test(Apple a) {
            return "green".equals(a.getColor());
        }
    });

    List<Apple> heavyApples = filterApple(inventory, new ApplePredicate() {
        @Override
        public boolean test(Apple a) {
            return a.getWeight() > 150;
        }
    });
    System.out.println("greenApples: " + greenApples);
    System.out.println("heavyApples: " + heavyApples);

    // greenApples: [Apple{color='green', weight=80}, Apple{color='green', weight=155}]
    // heavyApples: [Apple{color='green', weight=155}]
}
```


## Step6：Lambda表达式  

还能再简洁点么？  

当然，我们有`Lambda表达式`!  


``` java
public void test(){
    List<Apple> greenApples = filterApple(inventory, (Apple a) -> "green".equals(a.getColor()));
    System.out.println("greenApples: " + greenApples);

    // greenApples: [Apple{color='green', weight=80}, Apple{color='green', weight=155}]
}
```


## Step7：类型推断  

还能再简化么？  

当然，这个时候就要用到`类型推断`了。  

不过，我们先要介绍下`目标类型`。  

所谓`目标类型`，就是`Lambda表达式需要的类型`，举个例子：

上面`(Apple a) -> "green".equals(a.getColor())`的`目标类型`就是`ApplePredicate<Apple>`  

也就是说`Request`是`Apple`，`Response`是`boolean值`。  


因此，`Java编译器`可以从`目标类型`推断出`Lambda表达式的签名`：  


``` java
public void test(){
    List<Apple> greenApples = filterApple(inventory, a -> "green".equals(a.getColor()));
    System.out.println("greenApples: " + greenApples);

    // greenApples: [Apple{color='green', weight=80}, Apple{color='green', weight=155}]
}
```


## Step8：方法引用  

什么是`方法引用`？  

可以看做调用`特定方法`的`Lambda表达式`的一种`快捷写法`。  


``` java
public void test(){
    inventory.stream().map(a -> a.getColor()).forEach(c -> System.out.println("color: " + c));

    inventory.stream().map(Apple::getColor).forEach(c -> System.out.println("color: " + c));
    // color: green
    // color: green
    // color: red
}
```


## `Reference`  
- `《Java8实战》 - 2 通过行为参数化传递代码`  
