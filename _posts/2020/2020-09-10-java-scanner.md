---
layout: post
title: Java - Scanner用法
category: java-basic
tags: [java]
excerpt: Java Scanner Example  
---

本文介绍下`Java`中扫描类`Scanner`的常见用法：  

## Read Input  

首先，是基本的读取：  

``` java
try (Scanner sc = new Scanner(System.in)){

    System.out.println("Please enter your name: ");
    String name = sc.nextLine();
    System.out.println("name : " + name);

    System.out.println("Please enter your age: ");
    int age = sc.nextInt();
    System.out.println("age: " + age);
}

/** ------------------------*/
// Please enter your name: 
// Yao Frankie
// name : Yao Frankie
// Please enter your age: 
// 24
// age: 24
```

## Split Input  

`Scanner`的默认分隔符是空格，但是你可以通过`useDelimiter()`进行指定：  

``` java
String input = "1,2,3,4,5";
try(var sc = new Scanner(input).useDelimiter(",")){
    while (sc.hasNext()){
        System.out.printf(sc.next() + " ");
    }
}

/** ------------------------*/
// 1 2 3 4 5
```

## Read File  

然后，你还可以从文件中读取数据：  

``` java
try (var sc = new Scanner(new File("your_file_path"))){
    while (sc.hasNext()){
        System.out.println(sc.nextLine());
    }
} catch (FileNotFoundException e) {
    e.printStackTrace();
}
```

## Input Types   

八大基本类型，除了`char`，其他均有`nextXXX()`：  


``` java
try(var sc = new Scanner(System.in)){
    System.out.println("Please enter your name, age and salary: ");
    String name   = sc.nextLine();
    int    age    = sc.nextInt();
    double salary = sc.nextDouble();
    System.out.println("Name  : " + name);
    System.out.println("Age   : " + age);
    System.out.println("Salary: " + salary);
}

/** ------------------------*/
// Please enter your name, age and salary: 
// Yao Frankie
// 24 66.20
// Name  : Yao Frankie
// Age   : 24
// Salary: 66.2
```

## next() vs nextLine()    

先来看个例子：  


``` java
try(var sc = new Scanner(System.in)){
    System.out.println("Please enter you name: ");
    String name = sc.next();
    System.out.println("Name: " + name);
}

/** ------------------------*/
// Please enter you name: 
// Yao Frankie
// Name: Yao
```

我们的期望是返回`Yao Frankie`，但是只返回了`Yao`，因为`Scanner`默认以`空格`为分隔符。  
因此我们可以将分隔符设置为`换行符`：  

``` java
try(var sc = new Scanner(System.in)){
    System.out.println("Please enter you name: ");
    sc.useDelimiter("\n");
    String name = sc.next();
    System.out.println("Name: " + name);
}

/** ------------------------*/
// Please enter you name: 
// Yao Frankie
// Name: Yao Frankie
```

也可以直接使用`nextLine()`：  

``` java
try(var sc = new Scanner(System.in)){
    System.out.println("Please enter you name: ");
    String name = sc.nextLine();
    System.out.println("Name: " + name);
}

/** ------------------------*/
// Please enter you name: 
// Yao Frankie
// Name: Yao Frankie
```


## Reference  

- [Java Scanner examples](https://mkyong.com/java/java-scanner-examples/){:target="_blank"}  
- [Java User Input (Scanner)](https://www.w3schools.com/java/java_user_input.asp){:target="_blank"}  
- [Java Scanner Class](https://www.programiz.com/java-programming/scanner){:target="_blank"}  
