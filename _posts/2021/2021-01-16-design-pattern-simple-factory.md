---
layout: post
title: 设计模式 - 简单工厂模式
category: design-pattern
tags: [design-pattern]
excerpt: Simple Factory
---

## 概述  

在实际的软件开发过程中，有时需要创建一些来自`相同父类`的类的实例，为此我们可以专门定义一个类来负责创建这些类的实例。通常我们可以通过传入不同的参数来获得不同的对象，利用`Java`语言的特性，习惯上将创建这些实例的方法定义为`static`类型，因此该方法也称为静态工厂方法，所以`简单工厂模式`也称之为`静态工厂方法模式`。  


## 实例  

### 实例说明  

假设在一个订单系统中存在多种支付方式，如：支付宝、微信和现金支付，那么我们可以这样实现支付功能：  


``` 
public void pay(String type){

    if ("Alipay".equals(type)){
        // Pay it using Alipay.
    } else if ("Wechat".equals(type)){
        // Pay it using Wechat.
    } else if ("Cash".equals(type)){
        // Pay it using Cash.
    } else {
        // Sorry, we can't support this pay method!
    }
}
```


那么，这样写有什么问题？  

第一，在真实的场景中，相应的支付逻辑较为复杂，因此这个方法就会变得非常冗长且可读性很低。  
第二，如果需要新增一种支付类型，则需要改动该方法，因此会提升开发、测试和维护的难度。  


### 实例类图  

我们可以使用简单工厂模式重构以上支付逻辑，先来看下类图：  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/design_pattern_simple_factory.png)  


简单工厂模式包含三个角色：  

第一个是抽象产品角色，也就是上图中的`AbstractPay`抽象类，它是所创建的所有对象的父类，负责描述所有实例共有的行为。  

第二个是具体产品角色，也就是上图中的`AliPay`、`WechatPay`和`CashPay`类，所有创建的对象都是某个具体类的实例。  

最后一个是工厂类角色，也就是上图中的`PayMethodFactory`类，它负责实现创建所有实例的逻辑。  







### 实例代码及解释  

再来看下具体的代码：  

首先，我们需要抽象出支付这个行为，因此我们需要创建`AbstractPay `抽象类。  

```
public abstract class AbstractPay {
    public abstract void pay();
}
```

然后，我们分别创建多个具体的实现类来继承`AbstractPay `抽象类：  


```
public class AliPay extends AbstractPay{
    @Override
    public void pay() {
        System.out.println("Pay it using Alipay.");
    }
}

public class WechatPay extends AbstractPay{
    @Override
    public void pay() {
        System.out.println("Pay it using Wechat.");
    }
}

public class CashPay extends AbstractPay{
    @Override
    public void pay() {
        System.out.println("Pay it using cash.");
    }
}
```


接着，重点来了，我们需要创建一个特定的类来获得上面的不同实现类，注意其中方法返回的是`AbstractPay `抽象类：  


```
public class PayMethodFactory {

    public static AbstractPay getPayMethod(String type){
        if ("Alipay".equals(type)){
            return new AliPay();
        } else if ("Wechat".equals(type)){
            return new WechatPay();
        } else if ("Cash".equals(type)){
            return new CashPay();
        } else {
            throw new UnsupportedOperationException("Sorry, we can't support this pay method!");
        }
    }
}
```

最后，来看下测试代码：  

```
public class StaticFactoryTest {

    public static void main(String[] args) {

        AbstractPay wechatPay = PayMethodFactory.getPayMethod("Wechat");
        wechatPay.pay();
        AbstractPay alipay = PayMethodFactory.getPayMethod("Alipay");
        alipay.pay();
        AbstractPay others = PayMethodFactory.getPayMethod("others");
        others.pay();
    }
}


Pay it using Wechat.
Pay it using Alipay.
Exception in thread "main" java.lang.UnsupportedOperationException: Sorry, we can't support this pay method!
    at designpattern.creation.factory.staticfactory.PayFactory.getPayMethod(PayFactory.java:16)
    at designpattern.creation.factory.staticfactory.StaticFactoryTest.main(StaticFactoryTest.java:16)
```



## 优缺点  

接下来总结一下简单工厂模式的优缺点。  

### 优点  

第一，你只需要传入正确的参数，就可以获取你所需要的对象，不需要关心其创建细节。  

第二，实现了对象的创建和使用的分离。  

### 缺点  

第一，工厂类不够灵活，一旦需要新增新的具体产品，则需要修改工厂类的判断逻辑代码。  

第二，当产品数量较多时，工厂方法代码就会变得非常复杂。  


## Reference  

- 设计模式 - 第四章: 简单工厂模式  
