---
layout: post
title: Java - 泛型类型擦除
category: java-basic
tags: [spring, java]
excerpt: Java Generic Type-Erasure
---

## `类型擦除`  

困扰了我很久的问题：什么是`类型擦除`？  

当把一个`具有泛型信息的对象`赋给另一个`没有泛型信息的对象`时，所有`尖括号中的类型信息都会被扔掉`，这就是`类型擦除`。  

来看个例子：  

``` java
@Setter
@Getter
public class Apple<T extends Number> {

    private T size;

    public Apple(T size){
        this.size = size;
    }
}

@Test
public void typeEraseTest(){
    // Section 1
    Apple<Integer> integerApple = new Apple<>(8);
    Integer intSize = integerApple.getSize();

    // Section 2
    Apple<?> wildcardApple = integerApple;
    Number numsize1 = wildcardApple.getSize();

    // Section 3
    Apple rawApple  = integerApple;
    Number numSize2 = rawApple.getSize();
}
```

可以看到，在`Section 1`中`integerApple`对象的泛型是`Integer`，所以调用`getSize()`方法后也返回`Integer`。  

在`Section 2、3`中将`integerApple`对象赋给泛型为`通配符`、`不带泛型信息(原生类型)`的对象时，  
`integerApple`中的泛型信息就会被擦除，由于其泛型形参的上限是`Number`，  
所以后两者调用`getSize()`方法返回的是`Number`类型。  


## `Reference`
- `《疯狂Java讲义 - 9.5》 - 擦除和转换`  

