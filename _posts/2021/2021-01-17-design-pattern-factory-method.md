---
layout: post
title: 设计模式 - 工厂方法模式
category: design-pattern
tags: [design-pattern]
excerpt: Factory Method
---

## 概述  

工厂方法模式是简单工厂模式的延伸，它包含了简单工厂模式的优点，也弥补了它的不足，当添加新的具体产品对象时，不需要对已有系统做任何修改。工厂方法模式引入了抽象的工厂类，而将具体产品的创建过程封装在抽象工厂类的子类，也就是具体工厂类中。  

## 实例  

### 实例类图  


我们使用工厂方法模式对支付方式进行重构，先来看下类图：  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/design_pattern_factory_method.png)  

工厂方法模式包含四个角色：  

第一个是抽象产品角色，也就是上图中的`AbstractPay`抽象类，它是所有产品对象的父类，负责描述产品实例的共有行为。  

第二个是具体产品角色，也就是上图中的`AliPay`、`WechatPay`和`CashPay`类，它们实现了抽象产品接口，并且由专门的具体工厂创建。  

第三个是具体工厂角色，也就是上图中的`AlipayPayMethodFactory`、`WechatPayMethodFactory `和`CashPayMethodFactory`类，它们实现了抽象工厂中定义的方法，并返回一个具体产品类的实例。  

最后一个是抽象工厂角色，也就是上图中的`AbstractPayMethodFactory `抽象类，它声明了工厂方法，用于返回一个产品。  


### 实例代码及解释  

再来看下代码实现和解释：  

首先，和简单工厂模式一样，我们需要定义抽象产品类，和具体的产品类：  

```
public abstract class AbstractPay {
    public abstract void pay();
}

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

然后，我们需要定义抽象工厂类，注意它返回的是`抽象产品类`。  

```
public abstract class AbstractPayMethodFactory {
    public abstract AbstractPay getPayMethod();
}
```

接着，我们定义具体的工厂类，与具体产品类一一对应：  


```
public class AlipayPayMethodFactory extends AbstractPayMethodFactory {
    @Override
    public AbstractPay getPayMethod() {
        return new AliPay();
    }
}

public class WechatPayMethodFactory extends AbstractPayMethodFactory {
    @Override
    public AbstractPay getPayMethod() {
        return new WechatPay();
    }
}

public class CashPayMethodFactory extends AbstractPayMethodFactory {
    @Override
    public AbstractPay getPayMethod() {
        return new CashPay();
    }
}

```

最后，来看下测试代码：  

```
public class FactoryMethodTest {
    public static void main(String[] args) {

        AbstractPayMethodFactory alipayMethodFactory = new AlipayPayMethodFactory();
        AbstractPay alipay = alipayMethodFactory.getPayMethod();
        alipay.pay();

        AbstractPayMethodFactory cashPayMethodFactory = new CashPayMethodFactory();
        AbstractPay cashPay = cashPayMethodFactory.getPayMethod();
        cashPay.pay();
    }
}

Pay it using Alipay.
Pay it using cash.
```


## 优缺点  

### 优点  

第一，基于工厂和产品角色的`多态性`设计是工厂方法模式的关键。你可以选择不同的具体工厂类来创建不同的具体产品对象，而如何创建这个对象的细节则完全被封装在具体工厂类内部。  

第二，当需要添加新的产品时，无须修改抽象工厂和产品类，无须修改客户端，也无须修改其他的具体工厂和产品类，只需要添加一个具体的工厂和产品类即可。  


### 缺点  

第一，当添加新产品时，需要添加新的具体工厂和产品类，使得系统中类的数量成对增加，在一定程度上增加了系统的复杂度，并且有更多的类需要编译和运行，也会给系统带来一些额外的开销。  

第二，由于引入了抽象工厂类，因此也会增加系统的抽象性和理解程度。  


## Reference  

- 设计模式 - 第五章: 工厂方法模式  
