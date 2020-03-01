---
layout: post
title: Java - [面向对象]this
category: java-basic
tags: [spring, java]
excerpt: Java OOP this
---

本文梳理下`this`的几种使用场景：  


## 方法中的`this`  

假设定义一个`Dog`类，其中的`run()`方法需要调用它的`jump()`方法，应该怎么办？  

``` java
public class Dog {

    public void jump(){
        System.out.println("Start jumping");
    }

    public void run(){
        jump();
        System.out.println("Start running");
    }
}

@Test
void thisTest(){
    Dog dog = new Dog();
    dog.run();
    // Dog start jumping
    // Dog start running
}
```

有没有想过这样一个问题：  

`实例方法不是只能被对象调用吗？为什么run()方法中可以直接调用jump()？`  

其实上面的`jump()`，完整版应该是`this.jump()`，`Java`允许对象的一个成员直接调用另一个成员，因此可以省略`this`前缀。  

而这个省略的`this`其实代表调用`run()`这个方法的对象，就是`dog`。


再来一个特殊的场景：  

如果`方法中的局部变量`与`类中的实例成员变量`同名怎么办？  

``` java
public class Dog {

    private final String name = "puppy";

    public void getName(){
        String name = "abc";
        System.out.println("The name of dog is " + name);
    }
}

@Test
void thisTest(){
    Dog dog = new Dog();
    dog.getName();
    // The name of dog is abc
}
```

可以看到：`方法中的局部变量`覆盖了`类中的实例成员变量`。  


如果我就想访问`类实例成员变量`，应该怎么办？  

``` java
public class Dog {

    private final String name = "puppy";

    public void getName(){
        String name = "abc";
        System.out.println("The name of dog is " + this.name);
    }
}

@Test
void thisTest(){
    Dog dog = new Dog();
    dog.getName();
    // The name of dog is puppy
}
```

答案就是使用`this`。  

重新理解下这句话： `方法中的this代表调用该方法的对象`。  

那么上面的`this`在哪个方法中？  

谁调用的`getName()`方法？  

所以`dog.name`是什么？  

`puppy`  


## 构造器中的`this`  

构造器中的`this`代表什么？  

`构造器正在初始化的对象`  

举个例子：  

``` java
@Getter
public class Dog {

    private String name;

    public Dog(String name){
        this.name = name;
    }
}

@Test
void thisTest(){
    Dog puppy = new Dog("puppy");
    System.out.println("The name of dog is " + puppy.getName());
    // The name of dog is puppy
}
```

## `Reference`  
- `《疯狂Java讲义》 - 5.1.4 对象的this引用`  

