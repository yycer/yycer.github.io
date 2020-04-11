---
layout: post
title: Spring - 解析并注册BeanDefinition
category: Spring
tags: [spring]
excerpt: Spring - Parse And Register Bean Definition
---

## 解析BeanDefinition  

紧接着上一篇文章： [Spring - 容器的基本实现](http://yaoyichen.cn/spring/2020/04/11/spring-basic-implementation-of-container.html){:target="_blank"}  

``` java
/* DefaultBeanDefinitionDocumentReader line 189 */
private void parseDefaultElement(Element ele, BeanDefinitionParserDelegate 
    // import、alias、beans tags
    ...
    else if (delegate.nodeNameEquals(ele, BEAN_ELEMENT)) {
        processBeanDefinition(ele, delegate);
    }
    ...
}
```

这边主要介绍下`bean`标签：  


``` java
/**
 * DefaultBeanDefinitionDocumentReader line 305
 * Process the given bean element, parsing the bean definition
 * and registering it with the registry.
 */
protected void processBeanDefinition(Element ele, BeanDefinitionParserDelegate delegate) {
    BeanDefinitionHolder bdHolder = delegate.parseBeanDefinitionElement(ele);
    ...
    BeanDefinitionReaderUtils.registerBeanDefinition(bdHolder, getReaderContext().getRegistry());
    ...
}
```

`processBeanDefinition`这个方法很重要，它主要做了什么事情？  

首先，将`Element`元素解析成`BeanDefinition`。  

其次，注册`BeanDefinition`。  

当然还有两步操作：`decorateBeanDefinitionIfRequired`和`fireComponentRegistered`，有兴趣可以深入研究下。  

我们再来看一下`BeanDefinitionHolder`这个类：  

和`EncodedResource`一样，也是个封装类。  


``` java
public class BeanDefinitionHolder implements BeanMetadataElement {

    private final BeanDefinition beanDefinition;

    private final String beanName;

    @Nullable
    private final String[] aliases;
}
```

包含了`BeanDefinition`、`beanName`和`aliases`，这种命名、组装方式可以应用于实际项目中。  


接下来介绍下`bean`标签中的属性，主要解析工作在以下方法中进行：  


``` java
/* BeanDefinitionParserDelegate line 500 */
@Nullable
public AbstractBeanDefinition parseBeanDefinitionElement(
        Element ele, String beanName, @Nullable BeanDefinition containingBean) {
    ...
    if (ele.hasAttribute(CLASS_ATTRIBUTE)) {
        className = ele.getAttribute(CLASS_ATTRIBUTE).trim();
    }
    String parent = null;
    if (ele.hasAttribute(PARENT_ATTRIBUTE)) {
        parent = ele.getAttribute(PARENT_ATTRIBUTE);
    }

    try {
        AbstractBeanDefinition bd = createBeanDefinition(className, parent);

        parseBeanDefinitionAttributes(ele, beanName, containingBean, bd);
        bd.setDescription(DomUtils.getChildElementValueByTagName(ele, DESCRIPTION_ELEMENT));

        parseMetaElements(ele, bd);
        parseLookupOverrideSubElements(ele, bd.getMethodOverrides());
        parseReplacedMethodSubElements(ele, bd.getMethodOverrides());

        parseConstructorArgElements(ele, bd);
        parsePropertyElements(ele, bd);
        parseQualifierElements(ele, bd);

        bd.setResource(this.readerContext.getResource());
        bd.setSource(extractSource(ele));

        return bd;
    }
    ...
}
```

这边就不深入展开了，简单列下包含哪些属性：  


``` java
public static final String ID_ATTRIBUTE = "id";
public static final String NAME_ATTRIBUTE = "name";
public static final String CLASS_ATTRIBUTE = "class";
public static final String PARENT_ATTRIBUTE = "parent";
private static final String SINGLETON_ATTRIBUTE = "singleton";
public static final String SCOPE_ATTRIBUTE = "scope";
public static final String ABSTRACT_ATTRIBUTE = "abstract";
public static final String LAZY_INIT_ATTRIBUTE = "lazy-init";
public static final String AUTOWIRE_ATTRIBUTE = "autowire";
public static final String DEPENDS_ON_ATTRIBUTE = "depends-on";
public static final String AUTOWIRE_CANDIDATE_ATTRIBUTE ="autowire-candidate"; 
public static final String PRIMARY_ATTRIBUTE = "primary";
public static final String INIT_METHOD_ATTRIBUTE = "init-method";
public static final String DESTROY_METHOD_ATTRIBUTE = "destroy-method";
public static final String FACTORY_METHOD_ATTRIBUTE = "factory-method";
public static final String FACTORY_BEAN_ATTRIBUTE = "factory-bean";
public static final String LOOKUP_METHOD_ELEMENT = "lookup-method";
public static final String REPLACED_METHOD_ELEMENT = "replaced-method";
public static final String CONSTRUCTOR_ARG_ELEMENT = "constructor-arg";
public static final String PROPERTY_ELEMENT = "property";
public static final String QUALIFIER_ELEMENT = "qualifier";
```


## 注册BeanDefinition  

准确来说，需要注册两个东西：`BeanDefinition`和`alias`。  

``` java
/* BeanDefinitionReaderUtils line 160 */
public static void registerBeanDefinition(
        BeanDefinitionHolder definitionHolder, BeanDefinitionRegistry registry)
        throws BeanDefinitionStoreException {

    // Register bean definition under primary name.
    String beanName = definitionHolder.getBeanName();
    registry.registerBeanDefinition(beanName, definitionHolder.getBeanDefinition());

    // Register aliases for bean name, if any.
    String[] aliases = definitionHolder.getAliases();
    if (aliases != null) {
        for (String alias : aliases) {
            registry.registerAlias(beanName, alias);
        }
    }
}
```

先来说下注册`BeanDefinition`：  

``` java
/** Map of bean definition objects, keyed by bean name. */
private final Map<String, BeanDefinition> beanDefinitionMap = new ConcurrentHashMap<>(256);

/* DefaultListableBeanFactory line 908 */
@Override
public void registerBeanDefinition(String beanName, BeanDefinition beanDefinition)
        throws BeanDefinitionStoreException {

    ...
    this.beanDefinitionMap.put(beanName, beanDefinition);
    ...
}
```

重点就是将添加一个`entry`到`beanDefinitionMap`中，其中`key`是`beanName`，`value`就是`beanDefinition`。  


再来看下注册`alias`：  


``` java
/* SimpleAliasRegistry line 53*/
private final Map<String, String> aliasMap = new ConcurrentHashMap<>(16);

@Override
public void registerAlias(String name, String alias) {
    ...
    this.aliasMap.put(alias, name);
    ...
}
```

同样添加一个`entry`到`aliasMap`中，其中`key`为`alias`，而`value`为`beanName`。  


## `Reference`  
- `《Spring源码深度解析》 - 第三章 [bean标签的解析及注册]`
- [Spring 通过工厂方法(Factory Method)来配置bean](https://www.cnblogs.com/mochaMM/p/6925366.html){:target="_blank"}