---
layout: post
title: Spring - @Resource, @Autowire, @Inject
category: Spring
tags: [spring]
excerpt: Spring获得依赖的三种方式
---

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/DI_summary.png)


## 如何查看`Spring`容器中的所有`Bean`?  

``` Java
@Autowired
private ApplicationContext applicationContext;

public void printAllBeanNames(){
    String beanNames = Arrays.toString(applicationContext.getBeanDefinitionNames());
    String result = beanNames.replaceAll(",", "\n");
    System.out.println(result);
}
```

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/DI_print_all_beans.png)  

<br>

## 为什么要使用`@Resouce`, `@Autowire`, `@Inject`？  

我的理解是，通过`Spring`从容器中获得需要对象的三种方式。  

这样做的好处是，软件工程师不用操心如何创建对象，而是把这个任务全权交给`Spring`，当你需要用到某个对象时，直接从`Spring`容器中取即可。  

首先来看下这三种方式的概况：  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/DI_annotation_source_package.png)
<br>

## `Ambiguous Beans`  

先介绍下需要的接口与实现类：  

``` Java
public interface Party {

    /**
     * Just for test.
     */
    void printDefault();
}


@Component
public class Person implements Party {

    @Bean(value = "personBean")
    public Person buildPersonForTest(){
        return new Person();
    }

    @Override
    public void printDefault(){
        System.out.println("Hi Person.");
    }
}

@Component
public class Organization implements Party {
    @Override
    public void printDefault() {
        System.out.println("Hi Organization.");
    }
}

```


``` Java
/**
 * Ambiguous Beans.
 */
@Resource
private Party party;

@Autowired
private Party party;

@Inject
private Party party;

```

三种方式都会有如下问题：  

> Injection of resource dependencies failed;  
> nested exception is org.springframework.beans.factory.NoUniqueBeanDefinitionException:  
> No qualifying bean of type 'com.frankie.demo.interfaces.Party' available:  
> expected single matching bean but found 3: organization,person,personBean

也就是说，它只想要一个合适的对象，但是`Spring`容器中有三个，因此无法完成精准匹配。  

<br>
## `Field Name`  

``` Java
/**
 * Filed Type
 */
@Resource
private Party person;

@Autowired
private Party person;

@Inject
private Party person;

@Resource(name = "person")
private Party party;
```

以上四种情况都会正常获得`Person`类的实例。  

先说下`@Resource`，首先它会根据`field name`，也就是`person`尝试从`spring`容器中获得对应的`Bean`，很巧，容器中正好有这个`Bean`。  

然后说下`@Autowired、@Inject`，他两首先会根据`field type`也就是`party`，尝试从`spring`容器中获得对应的`Bean`，但是容器中并没有这个`Bean`，由于没有使用`@Qualifier`注解，因此尝试通过`filed name`获得`Bean`，bingo！  

至于第四种写法，虽然通过`field name`和`file type`都没有从`Spring`容器中获得对于的`Bean`，但是可以通过`@Resource`加上`name property`指定`bean name`的方式去获得。  


<br>
## `Field Type`  

``` Java
/**
 * Filed Type
 */
@Resource
private Person party;

@Autowired
private Person party;

@Inject
private Person party;
```

以上三种情况都会根据`field type`也就是`Person`，尝试从`Spring`容器中获得`person bean`，注意喔，根据`field type`去查找`Bean`时，用的是首字母小写的形式喔。  

<br>

## `Qualifier`  

``` Java
/**
 * Qualifier
 */

@Resource
@Qualifier(value = "personBean")
private Person frankie;

@Autowired
@Qualifier(value = "personBean")
private Person frankie;

@Inject
@Qualifier(value = "personBean")
private Person frankie;

@Resource(name = "personBean")
private Person frankie;

```

当软件工程师自定义`bean name`时，就有可能需要用到`@Qualifier`注解，有一种常见的场景：配置多数据源。  

当然，除了使用`@Qualifier`注解，我觉得使用`@Resource`加上`name property`的方式更加简洁明了。  

最后总结下，在真实的开发场景中，尽量避免构建`field name`与`bean name`的匹配关系，因为`filed name`随时可能会被你`Shift + F6`改变。  
一般情况下，直接使用`@Resource(name = "lowercase_class_name")`，  
当一个类或接口对应多个`Bean`时，使用`@Resource(name = "specified_bean_name")`。



## `Reference`  

- [SPRING INJECTION WITH @RESOURCE, @AUTOWIRED AND @INJECT](https://www.sourceallies.com/2011/08/spring-injection-with-resource-and-autowired/){:target="_blank"}
- [Wiring in Spring: @Autowired, @Resource and @Inject](https://www.baeldung.com/spring-annotations-resource-inject-autowire){:target="_blank"}

