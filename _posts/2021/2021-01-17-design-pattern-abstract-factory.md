---
layout: post
title: 设计模式 - 抽象工厂模式
category: design-pattern
tags: [design-pattern]
excerpt: Abstract Factory
---

## 概述  

抽象工厂方法也是常见的创建型设计模式之一，它比工厂方法模式的抽象程度更高。在工厂方法模式中具体工厂只需要生产一种具体的产品，而在抽象工厂模式中，具体工厂可以生产相关的一组具体产品，这样的一组产品称之为产品族，产品族中的每一个产品都属于某一个产品等级结构。  

产品等级结构有点难以理解，举个例子，假设有一个抽象类是电视机，其子类有海尔电视机、TCL电视机、小米电视机等，此时抽象电视机和具体品牌的电视机之间就构成了一个产品等级结构。  


## 实例  

### 实例说明  

一个电器工厂可以生产多种类型的电器，比如海尔工厂可以生产海尔电视机、海尔空调等，TCL工厂可以生成TCL电视机、TCL空调等，相同品牌的电器构成一个产品族，而相同类型的电器构成一个产品等级结构，现在使用抽象工厂模式模拟该场景。  

### 实例类图  

先来看下类图：  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/design_pattern_abstract_factory.png)  


抽象工厂模式包含四个角色：  

第一个是抽象工厂类角色，也就是上图中的`ElectronicsFactory`抽象类，它定义了两个抽象方法，一个返回电视机产品等级结构，另一个返回空调产品等级结构。  

第二个是具体工厂角色，也就是上图中的`HaierFactory `和`TCLFactory `类，它们实现了抽象工厂`ElectronicsFactory`类，分别构成一个产品族。  

第三个是具体产品角色，也就是上图中的`HaierTelevision`、`HaierAirConditioner`、`TCLTelevision`、和`TCLAirConditioner`类，它们实现了抽象产品`Television`和`AirConditioner`类定义的方法。  

最后一个是抽象产品类角色，也就是上图中的`Television`和`AirConditioner`类，它们定义了抽象的方法。  


### 实例代码及解释  

再来看下具体代码实现：  

首先，我们需要创建抽象工厂类：  

```
public abstract class ElectronicsFactory {
    public abstract Television produceTelevision();
    public abstract AirConditioner produceAirConditioner();
}
```

然后，需要创建具体的工厂类，生产一个产品族：  

```
public class HaierFactory extends ElectronicsFactory{
    @Override
    public Television produceTelevision() {
        return new HaierTelevision();
    }
    @Override
    public AirConditioner produceAirConditioner() {
        return new HaierAirConditioner();
    }
}

public class TCLFactory extends ElectronicsFactory{
    @Override
    public Television produceTelevision() {
        return new TCLTelevision();
    }
    @Override
    public AirConditioner produceAirConditioner() {
        return new TCLAirConditioner();
    }
}
```

接着，我们来定义具体的产品类：  


```
public class HaierTelevision implements Television{
    @Override
    public void play() {
        System.out.println("Play the TV using Haier.");
    }
}

public class HaierAirConditioner implements AirConditioner{
    @Override
    public void changeTemperature() {
        System.out.println("Change temperature using Haier.");
    }
}

public class TCLTelevision implements Television{
    @Override
    public void play() {
        System.out.println("Play the TV using TCL.");
    }
}

public class TCLAirConditioner implements AirConditioner{
    @Override
    public void changeTemperature() {
        System.out.println("Change temperature using TCL.");
    }
}
```

再来看下抽象产品的定义：  

```
public interface Television {
    void play();
}

public interface AirConditioner {
    void changeTemperature();
}
```

最后来看下测试代码：  

```

    public static void main(String[] args) {

        // 1. l need a Haier TV.
        ElectronicsFactory haierFactory = new HaierFactory();
        Television haierTV = haierFactory.produceTelevision();
        haierTV.play();

        // 2. l need a TCL air conditioner.
        ElectronicsFactory tclFactory = new TCLFactory();
        AirConditioner tclAirConditioner = tclFactory.produceAirConditioner();
        tclAirConditioner.changeTemperature();
    }
}

Play the TV using Haier.
Change temperature using TCL.
```

## 优缺点  

### 优点  

第一，实现产品的创建和使用分离。  

第二，可以通过具体工厂类创建一个`产品族`中的多个产品。  

### 缺点  

第一，增加新的产品等级结构很复杂，需要修改抽象工厂和所有的具体工厂类。  


## Reference  

- 设计模式 - 第六章: 抽象工厂模式  