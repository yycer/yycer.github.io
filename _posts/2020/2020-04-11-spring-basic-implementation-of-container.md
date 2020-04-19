---
layout: post
title: Spring - 容器的基本实现
category: Spring
tags: [spring]
excerpt: Spring - The basic implementation of container
---

## 简单例子  

``` java
@SpringBootTest
public class Chapter2Test {

    @Test
    public void basicTest(){
        XmlBeanFactory xmlBeanFactory = new XmlBeanFactory(new ClassPathResource("BeanFactoryTest.xml"));
        MyTestBean myTestBean = (MyTestBean) xmlBeanFactory.getBean("myTestBean");
        Assert.assertEquals(myTestBean.getTest(), "test");
    }
}

/* BeanFactoryTest.xml */
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="myTestBean" class="com.frankie.demo.beans.MyTestBean"/>

</beans>

@Getter
public class MyTestBean {

    private String test = "test";
}
```

相信大家对这个例子都很熟悉，但是有没有仔细思考过其内部到底发生了什么？`Spring`到底做了哪些事情？  

让我们一起探索`Spring`的内部吧~  


先来张概况图：  

<img src="https://yyc-images.oss-cn-beijing.aliyuncs.com/charpter2.png">

## `ClassPathResource`  

`Resource classPathResource = new ClassPathResource("BeanFactoryTest.xml");`  

可以看到`ClassPathResource`构造器接收一个文件路径，然后返回资源。  

这里提一下`URI`与`URL`的区别。  

首先`URI`的全称是：`Universal Resource Identifier`，翻译成：`统一资源标识符`，通过其唯一性来定位某一资源，就像每个人的身份证号码。  

而`URL`的全称是：`Universal Resource Locator`，翻译成：`统一资源定位符`，通过其路径定位某一资源，比如：`/地球/中国/上海/闵行区/某小区/1号楼/101室/张三`。  

Thanks [@daixinye](https://www.zhihu.com/question/21950864){:target="_blank"}  


## `loadBeanDefinitions`  

``` java
/* XmlBeanFactory line 77 */
public XmlBeanFactory(Resource resource, BeanFactory parentBeanFactory) throws BeansException {
    super(parentBeanFactory);
    this.reader.loadBeanDefinitions(resource);
}

/* XmlBeanDefinitionReader line 304 */
@Override
public int loadBeanDefinitions(Resource resource) throws BeanDefinitionStoreException {
    return loadBeanDefinitions(new EncodedResource(resource));
}

/* XmlBeanDefinitionReader line 315 */
public int loadBeanDefinitions(EncodedResource encodedResource) throws BeanDefinitionStoreException {
    ...
    try {
        InputStream inputStream = encodedResource.getResource().getInputStream();
        try {
            InputSource inputSource = new InputSource(inputStream);
            if (encodedResource.getEncoding() != null) {
                inputSource.setEncoding(encodedResource.getEncoding());
            }
            return doLoadBeanDefinitions(inputSource, encodedResource.getResource());
        }
        finally {
            inputStream.close();
        }
    }
    ...
}
```

上面代码中用到了一个封装类：`EncodedResource`，为什么要用它呢？  

先来看看其内部包含什么：  

``` java
public class EncodedResource implements InputStreamSource {

    private final Resource resource;

    @Nullable
    private final String encoding;

    @Nullable
    private final Charset charset;
    ...
}
```

可以看到，是出于`编码`和`字符集`的考虑。  




## `doLoadBeanDefinitions`  

``` java
/* XmlBeanDefinitionReader line 389 */
protected int doLoadBeanDefinitions(InputSource inputSource, Resource resource)
            throws BeanDefinitionStoreException {

    try {
        Document doc = doLoadDocument(inputSource, resource);
        int count = registerBeanDefinitions(doc, resource);
        if (logger.isDebugEnabled()) {
            logger.debug("Loaded " + count + " bean definitions from " + resource);
        }
        return count;
    }
    ...
}
```

`doLoadBeanDefinitions`是一个非常重要的方法！  
`doLoadBeanDefinitions`是一个非常重要的方法！  
`doLoadBeanDefinitions`是一个非常重要的方法！  

这里面主要做了两件事情：  

第一：将`Resource`转换成`Document`。  
第二：注册`BeanDefinition`。  


## `doLoadDocument`  

``` java
/* XmlBeanDefinitionReader line 434 */
protected Document doLoadDocument(InputSource inputSource, Resource resource) throws Exception {
    return this.documentLoader.loadDocument(inputSource, getEntityResolver(), this.errorHandler,
            getValidationModeForResource(resource), isNamespaceAware());
}
```

我们先来看下`getValidationModeForResource`这个方法：  

它的作用是判断资源的模式。  

那么有哪几种模式？  

答案： `XSD`和`DTD`。  

先来说下`XSD`，它的全称是`XML Sechma Definition`。  

举个例子：  

``` java
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="myTestBean" class="com.frankie.demo.beans.MyTestBean"/>
</beans>
```

再来看下`DTD`，它的全称是`Docment Type Definition`。  

也举个例子：  

``` java
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE note SYSTEM "Note.dtd">
<note>
<to>Tove</to>
<from>Jani</from>
<heading>Reminder</heading>
<body>Don't forget me this weekend!</body>
</note>
```

Thanks [@w3schools](https://www.w3schools.com/xml/xml_dtd.asp){:target="_blank"}  

我们来看下`Spring`是依据什么来区别不同的模式：  

``` java
/*XmlValidationModeDetector line 127 */
private boolean hasDoctype(String content) {
    return content.contains(DOCTYPE);
}

private static final String DOCTYPE = "DOCTYPE";
```

很简单，`XML`中包含`DOCTYPE`就是`DTD`模式，否则就是`XSD`!  


继续来看，如何将`Resource`转换成`Document`：  

``` java
/* DefaultDocumentLoader line 69 */
@Override
public Document loadDocument(InputSource inputSource, EntityResolver entityResolver,
        ErrorHandler errorHandler, int validationMode, boolean namespaceAware) throws Exception {

    DocumentBuilderFactory factory = createDocumentBuilderFactory(validationMode, namespaceAware);
    if (logger.isTraceEnabled()) {
        logger.trace("Using JAXP provider [" + factory.getClass().getName() + "]");
    }
    DocumentBuilder builder = createDocumentBuilder(factory, entityResolver, errorHandler);
    return builder.parse(inputSource);
}
```


## `registerBeanDefinitions`  


``` java
/* XmlBeanDefinitionReader line 511 */
public int registerBeanDefinitions(Document doc, Resource resource) throws BeanDefinitionStoreException {
    BeanDefinitionDocumentReader documentReader = createBeanDefinitionDocumentReader();
    int countBefore = getRegistry().getBeanDefinitionCount();
    documentReader.registerBeanDefinitions(doc, createReaderContext(resource));
    return getRegistry().getBeanDefinitionCount() - countBefore;
}

/* DefaultBeanDefinitionDocumentReader line 121*/
protected void doRegisterBeanDefinitions(Element root) {
    ...
    preProcessXml(root);
    parseBeanDefinitions(root, this.delegate);
    postProcessXml(root);
    ...
}

/* DefaultBeanDefinitionDocumentReader line 168*/
protected void parseBeanDefinitions(Element root, BeanDefinitionParserDelegate delegate) {
    ...
    parseDefaultElement(ele, delegate);
    ...
    delegate.parseCustomElement(ele);
    ...
}
```

至此，容器的基本实现逻辑就结束了，下一篇文章主要介绍下如何解析默认元素`parseDefaultElement`。  


## `Reference`  
- `《Spring源码深度解析》 - 第二章 [容器的基本实现]`  

