---
layout: post
title: Java反射浅析
category: java-basic
tags: [java-basic]
excerpt: 梳理Java反射机制
---
![反射概述](http://px8rn4o1y.bkt.clouddn.com/%E5%8F%8D%E5%B0%84.png
)

## 为什么要用反射？
> 程序有时需要在运行时，获得、修改对象的信息。  

<br>
## 如何获得Class对象？
> 三种方式获得Class对象  

- 1.&nbsp;使用Class`类`的forName("全限定名")静态方法
- 2.&nbsp;调用某个`类`的class属性
- 3.&nbsp;调用某个`对象`的getClass()方法。

> 单元测试  

```java
    @Test
    public void getClassTest() throws ClassNotFoundException {
        // 方式1: 使用Class类的forName("全限定名")静态方法
        Class<?> person = Class.forName("com.frankie.demo.reflection.Person");

        // 方式2: 调用某个类的class属性
        Class<Person> person2 = Person.class;

        // 方式3: 调用某个对象的getClass()方法。
        Person p = new Person(2, "1");
        Class<? extends Person> person3 = p.getClass();
    }
```

<br>
## 如何生成并操作Class对象？

> 首先，通过Class对象的构造器方法生成对象，然后调用相关属性、方法来改变对象的状态或行为。  

```java
    @Test
    public void playObjectUsingReflectionTest()
            throws NoSuchMethodException, IllegalAccessException,
            InvocationTargetException, InstantiationException {

        Class<Person> personClass = Person.class;
        Person person = personClass.getConstructor(int.class, String.class).newInstance(37, "frankie");
        System.out.println("Id   = " + person.getId()   + " 调用对象的getId()方法");
        System.out.println("Name = " + person.getName() + " 调用对象的getName()方法");

        Method setIdMethod = personClass.getMethod("setId", int.class);
        Method getIdMethod = personClass.getMethod("getId");
        setIdMethod.invoke(person, 10);
        System.out.println("Id   = 10 通过反射方式执行setId()方法");
        int id = (int)getIdMethod.invoke(person);
        System.out.println("Id   = " + id + " 通过反射方式执行getId()方法");

        Method setNameMethod = personClass.getMethod("setName", String.class);
        Method getNameMethod = personClass.getMethod("getName");
        setNameMethod.invoke(person, "yyc");
        System.out.println("Name = yyc 通过反射方式执行setName()方法");
        String name = (String)getNameMethod.invoke(person);
        System.out.println("Name = " + name + " 通过反射方式执行getName()方法");

        Method changeNameMethod = personClass.getMethod("changeName", String.class);
        changeNameMethod.invoke(person, "asan");
        System.out.println("Name = asan" + " 通过反射方式执行Person类中的changeName()方法");
        String changedName = (String)getNameMethod.invoke(person);
        System.out.println("Name = " + changedName + " 通过反射方式执行getName()方法");
    }

    ---------------------Console information-------------------------------

    Id   = 37      调用对象的getId()方法
    Name = frankie 调用对象的getName()方法
    Id   = 10   通过反射方式执行setId()方法
    Id   = 10   通过反射方式执行getId()方法
    Name = yyc  通过反射方式执行setName()方法
    Name = yyc  通过反射方式执行getName()方法
    Name = asan 通过反射方式执行Person类中的changeName()方法
    Name = asan 通过反射方式执行getName()方法

```
<br>
## 参考
- 《疯狂Java讲义》 - 第十八章(类加载机制与反射)