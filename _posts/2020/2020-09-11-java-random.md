---
layout: post
title: Java - 生成随机数
category: java-basic
tags: [java]
excerpt: Java Generate Random Number 
---

本文总结一下如何通过`Java`生成随机数，如：  

第一个问题： 如何生成一个`[0, 100)`的整数？  
第二个问题： 如何生成一个`[50.0, 100.0)`的浮点数？  


## Random  

先来解决第一个问题：  

``` java
var random = new Random();
for (int i = 0; i < 3; i++){
    System.out.println("nextInt() = " + random.nextInt(100));
}

/** ------------------------*/
// nextInt() = 51
// nextInt() = 80
// nextInt() = 30
```

再来看下第二个问题：  

``` java
double min = 50.0, max = 100.0;
var random = new Random();
for (int i = 0; i < 3; i++){
    double d = random.nextDouble() * (max - min + 1) + min;
    System.out.println("[50.0, 100.0) = " + d);
}

/** ------------------------*/
// [50.0, 100.0) = 60.84508904919827
// [50.0, 100.0) = 82.38206370834861
// [50.0, 100.0) = 51.3422651357679
```

还可以使用`Random`类在`Java8`中添加的新特性： `doubles()`   

``` java
double min = 50.0, max = 100.0;
var random = new Random();
for (int i = 0; i < 3; i++){
    double d = random.doubles(min, max).findAny().getAsDouble();
    System.out.println("[50.0, 100.0) = " + d);
}

/** ------------------------*/
// [50.0, 100.0) = 88.7749323881711
// [50.0, 100.0) = 66.15906089543594
// [50.0, 100.0) = 59.42705244256021
```


## Math.random()  


首先，我们需要明确`Math.random()`的返回值是一个`[0.0, 1.0)`且类型为`double`的数字。    

同样，我们来解决第一个问题：  

``` java
int min = 0;
int max = 100;

for (int i = 0; i < 3; i++){
    int ans = (int) (Math.random() * (max - min + 1) + min);
    System.out.println("[0, 100) = " + ans);
}

/** ------------------------*/
// [0, 100) = 97
// [0, 100) = 10
// [0, 100) = 69
```

再来看下第二个问题：  

``` java
int min = 50;
int max = 100;

for (int i = 0; i < 3; i++){
    double d = Math.random() * (max - min + 1) + min;
    System.out.println("[50.0, 100.0) = " + d);
}

/** ------------------------*/
// [50.0, 100.0) = 58.222150040843616
// [50.0, 100.0) = 51.72442492530528
// [50.0, 100.0) = 72.4776027813157
```

## ThreadLocalRandom    

老规矩，先来解决第一个问题：  


``` java
for (int i = 0; i < 3; i++){
    int ans = ThreadLocalRandom.current().nextInt(100);
    System.out.println("[0, 100) = " + ans);
}

/** ------------------------*/
// [0, 100) = 29
// [0, 100) = 53
// [0, 100) = 23
```

再来看下第二个问题：  

``` java
int min = 50, max = 100;
for (int i = 0; i < 3; i++){
    double d = ThreadLocalRandom.current().nextDouble(min, max);
    System.out.println("[50.0, 100.0) = " + d);
}

/** ------------------------*/
// [50.0, 100.0) = 90.090332923636
// [50.0, 100.0) = 80.59064520230083
// [50.0, 100.0) = 69.10384148888186
```

## Reference    

- [How to generate random numbers in Java](https://www.educative.io/edpresso/how-to-generate-random-numbers-in-java){:target="_blank"}  
- [Random Number Generation in Java](https://dzone.com/articles/random-number-generation-in-java){:target="_blank"}  
- [Generating Random Numbers in a Range in Java](https://www.baeldung.com/java-generating-random-numbers-in-range){:target="_blank"}  