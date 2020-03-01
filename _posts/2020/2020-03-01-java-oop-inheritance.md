---
layout: post
title: Java - [面向对象]继承
category: java-basic
tags: [spring, java]
excerpt: Java OOP Inheritance
---

本文整理下继承几个有意思的地方。  

## `super`  

``` java
public class Parent {

    public void test(){
        System.out.println("Start test() in Parent");
    }
}

public class Child extends Parent {

    @Override
    public void test(){
        System.out.println("Start test() in Child");
    }
}

@Test
void superTest(){
    Child child = new Child();
    child.test(); // Start test() in Child
}
```

很明显，调用的是`Child`类中重写的`test()`方法。  

那么，如何在子类中调用父类的重写方法？  

答案就是: `super`  

``` java
public class Child extends Parent {

    @Override
    public void test(){
        super.test();
        // System.out.println("Start test() in Child");
    }
}

@Test
void superTest(){
    Child child = new Child();
    child.test(); // Start test() in Parent
}
```


## 继承树间构造器的启动顺序  

``` java
public class Creature {

    public Creature(){
        System.out.println("Default constructor in Creature.");
    }
}

public class Animal extends Creature{

    public Animal(String name){
        System.out.println("The name of animal is " + name);
    }

    public Animal(String name, int age){
        this(name);
        System.out.println(String.format("Age is %d", age));
    }
}

public class Wolf extends Animal {

    public Wolf() {
        super("Wolf", 3);
        System.out.println("Default constructor in Wolf.");
    }
}

@Test
void inheritTreeCreationSeqTest(){
    new Wolf();
}
```

来猜下构造器的执行顺序。  


``` java
// Default constructor in Creature.
// The name of animal is Wolf
// Age is 3
// Default constructor in Wolf.
```

## `Reference`
- `《疯狂Java讲义》 - 5.6 类的继承`  