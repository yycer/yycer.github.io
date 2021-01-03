---
layout: post
title: UML - 类图
category: 
tags: []
excerpt: UML - Class Diagram
---


Update at 2021_0103

阅读了刘伟主编的《设计模式》，对类图有了进一步的理解。  

首先，来看一下类的UML图表示：  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/uml_employee.png)  

补充一点：`下划线`代表静态成员变量和方法。  


再来看来类与类之间的关系，主要分为以下4种：  
 
- Association  
- Dependency  
- Generalization  
- Realization  


### Association  

关联是类与类之间最常用的一种关系，用于表示一类对象与另一类对象之间有联系，如汽车和轮胎、师傅和徒弟、老师和学生等。  

可以细分为以下6种：  

第一种：双向关联。  

举个例子： 顾客`(Customer)`可以购买商品`(Product)`，同时该商品与该顾客也产生关联，如下图所示：  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/uml_two_way_association.png)  

再来看下代码实现：  

``` 
public class Customer {

    private Product[] products;
}

public class Product {

    private Customer customer;
}
```


第二种，单向关联。  

单向关联用带箭头的实线表示，例如，顾客拥有地址，则`(Customer)`类与`(Address)`类具有单向关联的关系，如下图所示：  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/uml_one_way_association.png)  

再来看下代码实现：  

```
public class Customer {

    private Address address;
}

public class Address {

}

```


第三种，自关联。  

有时会存在一些类的属性对象类型为该类本身，这种特殊的关联关系称为自关联。例如，一个节点类`(Node)`的成员又是节点对象，如下图所示：  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/uml_self_association.png)  

再来看下代码实现：  

```
public class Node {

    private Node subNode;
}
```


第四种，多重性关联。  

多重性关联表示一个类的对象与另一个类的对象连接的个数，类的对象之间存在多种多重性关联关系，具体如下:  

- `1..1`: 表示另一个类的一个对象只与一个该类对象有关系。  
- `0..*`: 表示另一个类的一个对象与零个或多个该类对象有关系。  
- `1..*`: 表示另一个类的一个对象与一个或多个该类对象有关系。  
- `0..1`: 表示另一个类的一个对象与零个或一个该类对象有关系。  
- `m..n`: 表示另一个类的一个对象与最少m个、最多n个该类对象有关系。(m<=n)  

举个例子，一个界面`(Form)`可以拥有零个或多个按钮`(Button)`，但是一个按钮只能属于一个界面，类图如下所示：  


![](https://yyc-images.oss-cn-beijing.aliyuncs.com/uml_multiple_association.png)  

再来看下代码实现：  

```
public class Form {

    private Button[] buttons;
}

public class Button {

}
```



第五种，聚合关系。  

聚合关系`(Aggregation)`表示整体与部分的关系，如一台计算机包含内存条、显卡、CPU等部分。在聚合关系中，成员类是整体类的一部分，但是成员对象也可以脱离整体对象而独立存在。在`UML`中，聚合关系用带空心菱形的直线表示，例如，汽车发动机`(Engine)`是汽车`(Car)`的组成部分，但是汽车发动机也可以独立存在，如下图所示：  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/uml_aggregation.png)  

再来看下代码实现：  

```
public class Car {

    private Engine engine;

    public Car(Engine engine) {
        this.engine = engine;
    }

    public void setEngine(Engine engine) {
        this.engine = engine;
    }
}

public class Engine {

}
```



第六种，组合关系。  

组合关系`(Composition)`也用于表示类之间整体和部分的关系，但是组合关系中整体和部分具有统一的生存期。一旦整体对象不存在，部分对象也将不复存在，因此整体和部分对象具有同生共死的关系。在`UML`中，用带实心菱形的直线表示。例如，人的头`(Head)`与嘴巴`(Mouth)`，嘴巴是头的组成部分之一，因此头与嘴巴是组合关系，如下图所示：  


![](https://yyc-images.oss-cn-beijing.aliyuncs.com/uml_composition.png)  

再来看下代码实现：  

```
public class Head {

    private Mouth mouth;

    public Head(){
        mouth = new Mouth();
    }
}

public class Mouth {

}
```


### Dependency  

依赖关系是一种使用关系。大多数情况下，依赖关系体现在某个类的方法使用另一个类的对象作为参数。在`UML`中，用带箭头的虚线表示，由依赖的一方指向被依赖的一方。例如，驾驶员开车，在`(Driver)`类的`drive()`方法中将`(Car)`类型的对象`car`作为一个参数传递，以便在`drive()`方法中能调用`car`的`move()`方法，如下图所示：  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/uml_dependency.png)  

再来看下代码实现：  

```
public class Driver {

    public void drive(Car car){
        car.move();
    }
}

public class Car {

    public void move(){

    }
}
```


### Generalization  

泛化关系也就是继承关系，用于描述父类与子类的关系。在`UML`中，泛化用带空心三角形的直线表示。例如，我们定义一个`(Person)`类，他们有姓名和年龄的属性，同时他们还可以移动和说话，然后，我们可以创建一个`(Student)`类来继承`(Person)`类，每个学生都有自己的学号，并且他们的主要任务就是学习。接着，我们还可以创建一个`(Teacher)`类来继承`(Person)`类，每个老师也有自己的编号，而他们的任务就是教学生知识。如下图所示：  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/uml_generalization.png)  

再来看下代码实现：  

```
public class Person {

    protected String name;
    protected int age;

    public void move(){

    }

    public void say(){

    }
}


public class Student extends Person {

    private String studentNo;

    public void study(){

    }
}

public class Teacher extends Person {

    private String teacherNo;

    public void teach(){

    }
}
```





### Realization

类和接口还存在一种实现关系，类中的方法实现了接口中声明的方法。在`UML`中，类与接口之间用带空心三角形的虚线来表示。例如，我们定义一个交通工具接口`(Vehicle)`,其中有一个抽象方法`move()`，在类`(Ship)`和`(Bus)`中各自实现了该方法，如下图所示：  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/uml_realization.png)  

再来看下代码实现：  

```
public interface Vehicle {

    void move();
}

public class Ship implements Vehicle {

    @Override
    public void move() {

    }
}


public class Bus implements Vehicle{

    @Override
    public void move() {

    }
}
```






本文总结一下`UML`类图的用法，主要包含基本结构、可见性、类关系和多重性，如图：  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/UML_class_diagram.png)  


## Basic    

首先，我们先来看下类图的定义：  

> In software engineering, a class diagram in the Unified Modeling Language (UML) is a type of static structure diagram that describes the structure of a system by showing the system's classes, their attributes, operations (or methods), and the relationships among objects.  

`类图`是`UML`中的一种`静态结构图`，它的作用是通过展示系统中的类、及其成员变量和方法、对象之间的关系来描述整个系统的结构。  

然后，我们来看下`类图`的基本结构：  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/class_diagram_demo.png)  

主要由以下三部分组成：  

- 类名  
- 类成员变量  
- 类方法  


## Visibility    

在上图中，成员变量和方法的最左侧有`-`和`+`符号，那么它们有什么含义呢？  

它们其实指定了类成员的可见性，与`Java`类似，主要有以下四种：  

- `~`: Package / Default  
- `+`: Public  
- `-`: Private  
- `#`: Protected  


## Relationships    

再来看下类之间的关系：   

### Inheritance  

表示两个类拥有继承关系：  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/inheritance.png)  


### Association  

表示类之间的存在某种关系，如：航班与飞机、乘客与地铁等。  


![](https://yyc-images.oss-cn-beijing.aliyuncs.com/association.png)  


### Aggregation  

`Aggregation`是`Association`中更为具体的一种表示方式。   

当一个类作为其他类的集合或容器时，就会发生`Aggregation`，   

但是被包含的类的生命周期并不会依赖于该容器，所以即使容器类被销毁，被包含的类仍存在。   

比如： 图书馆与书，即使图书馆不存在了(翻修)，书仍可以存放在仓库中。   

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/aggregation.png)  


### Composition    

`Composition`与`Aggregation`类似，区别是：  

当容器类被销毁时，它的被包含的类也会销毁。  

比如： 一间房子中有卧室、厨房、阳台等，一旦房子被拆，那么其组成部分均被销毁。  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/composition.png)  


## Multiplicity    


最后介绍下类与类之间的数量匹配关系，一般有以下五种：  

- `0..1`: 零或一  
- `n`: 固定数量   
- `0..*`: 零或多个  
- `1..*`: 一或多个  
- `m..n`: 指定范围    

比如：   

- 一笔订单只能有一笔订单详情记录。  
- 一个用户可以有零或多笔订单。  



## Reference    
- [UML Class Diagram Tutorial](https://www.youtube.com/watch?v=UI6lqHOVHic){:target="_blank"}  
- [Class diagram](https://en.wikipedia.org/wiki/Class_diagram){:target="_blank"}  
- [UML Class Diagram Relationships Explained with Examples](https://creately.com/blog/diagrams/class-diagram-relationships/){:target="_blank"}  
- [lucidchart](https://app.lucidchart.com/){:target="_blank"}  


