---
layout: post
title: Spring - 加载Bean
category: Spring
tags: [spring]
excerpt: Spring - Get And Create Bean
---

这篇文章主要是想整理下`加载Bean`的整体流程，主要分为三大块：  

- 前需准备： `FactoryBean接口`  
- 关键方法： `getBean()`、`createBean()`  
- 循环依赖： 构造器、`Setter`、`Prototype`  

让我们开始这段旅程吧。  

## FactoryBean  

先来看下`FactoryBean`接口的内部结构：  

``` java
public interface FactoryBean<T> {

    String OBJECT_TYPE_ATTRIBUTE = "factoryBeanObjectType";

    /**
     * Return an instance (possibly shared or independent) of the object
     * managed by this factory. 
     */
    @Nullable
    T getObject() throws Exception;

    /**
     * Return the type of object that this FactoryBean creates,
     * or null if not known in advance.
     */
    @Nullable
    Class<?> getObjectType();

    default boolean isSingleton() {
        return true;
    }

}
```

你可以在`getObject()`方法创建任何你想返回的对象。  

先有个印象，我会在下面举个例子。  

下面来看下两个核心方法`getBean()和createBean()`的概况：  


![](https://yyc-images.oss-cn-beijing.aliyuncs.com/get-bean-process.png) 

## getBean()  

我们来到了第一个核心方法，先做一些准备工作：  

出于篇幅考虑，我会省掉一些非关键代码。  


``` java
@Setter
@Getter
public class Car {
    private String brand;
    private int    speed;
    private double price;

    @Override
    public String toString() {
        return ...
    }
}

@Setter
@Getter
public class CarFactoryBean implements FactoryBean<Car> {

    private String carInfo;

    @Override
    public Car getObject() throws Exception {
        String[] carInfo = this.carInfo.split(";");
        Car car = new Car();
        car.setBrand(carInfo[0]);
        car.setSpeed(Integer.parseInt(carInfo[1]));
        car.setPrice(Double.parseDouble(carInfo[2]));
        return car;
    }

    @Override
    public Class<?> getObjectType() {
        return Car.class;
    }
}

<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans  
       http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="car" class="com.frankie.demo.utils.CarFactoryBean">
        <property name="carInfo" value="tesla;400;200000"/>
    </bean>
</beans>
```

首先来看下`doGetBean()`方法的关键方法：  

``` java
/* AbstractBeanFactory line 242 */
@SuppressWarnings("unchecked")
protected <T> T doGetBean(final String name, @Nullable final Class<T> requiredType,
        @Nullable final Object[] args, boolean typeCheckOnly) throws BeansException {

    final String beanName = transformedBeanName(name);

    // Eagerly check singleton cache for manually registered singletons.
    Object sharedInstance = getSingleton(beanName);
    ...
    bean = getObjectForBeanInstance(sharedInstance, name, beanName, null);
        ...
        try {
            final RootBeanDefinition mbd = getMergedLocalBeanDefinition(beanName);
            ...
            // Create bean instance.
            if (mbd.isSingleton()) {
                sharedInstance = getSingleton(beanName, () -> {
                    try {
                        return createBean(beanName, mbd, args);
                    }
                    ...
                });
                bean = getObjectForBeanInstance(sharedInstance, name, beanName, mbd);
            ...
        }
    ...
    return (T) bean;
}
```

#### 1、transformedBeanName(name)  

``` java
Car car = (Car) context.getBean("car");
```

虽然你输入的`beanName = car`，但由于它是一个`FactoryBean`，所以`beanName`实际上是`&car`。  


``` java
/**
 * AbstractBeanFactory line 242
 * name = &car
 */
protected String transformedBeanName(String name) {
    return canonicalName(BeanFactoryUtils.transformedBeanName(name));
}

String FACTORY_BEAN_PREFIX = "&";

/* BeanFactoryUtils line 81 */
public static String transformedBeanName(String name) {
    Assert.notNull(name, "'name' must not be null");
    if (!name.startsWith(BeanFactory.FACTORY_BEAN_PREFIX)) {
        return name;
    }
    return transformedBeanNameCache.computeIfAbsent(name, beanName -> {
        do {
            beanName = beanName.substring(BeanFactory.FACTORY_BEAN_PREFIX.length());
        }
        while (beanName.startsWith(BeanFactory.FACTORY_BEAN_PREFIX));
        return beanName;
    });
}
```

所以当遇到`FactoryBean`时，`transformedBeanName()`方法的作用就是去除`beanName`的前缀`&`。  


#### 2、getSingleton(beanName)  

先尝试从缓存中获取`bean`实例。  


``` java
/** Cache of singleton objects: bean name to bean instance. */
private final Map<String, Object> singletonObjects = new ConcurrentHashMap<>(256);

/** Names of beans that are currently in creation. */
private final Set<String> singletonsCurrentlyInCreation =
        Collections.newSetFromMap(new ConcurrentHashMap<>(16));

/** Cache of early singleton objects: bean name to bean instance. */
private final Map<String, Object> earlySingletonObjects = new HashMap<>(16);

/** Cache of singleton factories: bean name to ObjectFactory. */
private final Map<String, ObjectFactory<?>> singletonFactories = new HashMap<>(16);

/* DefaultSingletonBeanRegistry line 176 */
@Nullable
protected Object getSingleton(String beanName, boolean allowEarlyReference) {
    Object singletonObject = this.singletonObjects.get(beanName);
    if (singletonObject == null && isSingletonCurrentlyInCreation(beanName)) {
        synchronized (this.singletonObjects) {
            singletonObject = this.earlySingletonObjects.get(beanName);
            if (singletonObject == null && allowEarlyReference) {
                ObjectFactory<?> singletonFactory = this.singletonFactories.get(beanName);
                if (singletonFactory != null) {
                    singletonObject = singletonFactory.getObject();
                    this.earlySingletonObjects.put(beanName, singletonObject);
                    this.singletonFactories.remove(beanName);
                }
            }
        }
    }
    return singletonObject;
}
```

#### 3、getObjectForBeanInstance(sharedInstance, name, beanName, null)  

当你使用`FactoryBean`时，就会触发这个方法，作用是执行`getObject()`方法，返回相应对象。  


``` java
/**
 * AbstractBeanFactory line 260
 * sharedInstance = CarFactoryBean@4619
 * name = beanName = car
 */
bean = getObjectForBeanInstance(sharedInstance, name, beanName, null);


/* AbstractAutowireCapableBeanFactory line 1258 */
@Override
protected Object getObjectForBeanInstance(
        Object beanInstance, String name, String beanName, @Nullable RootBeanDefinition mbd) {
    ...
    return super.getObjectForBeanInstance(beanInstance, name, beanName, mbd);
}

/* AbstractBeanFactory line 1778 */
protected Object getObjectForBeanInstance(
            Object beanInstance, String name, String beanName, @Nullable RootBeanDefinition mbd) {
    ...
    if (object == null) {
        // Return bean instance from factory.
        FactoryBean<?> factory = (FactoryBean<?>) beanInstance;
        // Caches object obtained from FactoryBean if it is a singleton.
        if (mbd == null && containsBeanDefinition(beanName)) {
            mbd = getMergedLocalBeanDefinition(beanName);
        }
        boolean synthetic = (mbd != null && mbd.isSynthetic());
        object = getObjectFromFactoryBean(factory, beanName, !synthetic);
    }
    return object;
}

/* FactoryBeanRegistrySupport line 96 */
protected Object getObjectFromFactoryBean(FactoryBean<?> factory, String beanName, boolean shouldPostProcess) {
    ...
    object = doGetObjectFromFactoryBean(factory, beanName);
    ...
    beforeSingletonCreation(beanName);
    ...
    afterSingletonCreation(beanName);
    ...
    return object;
    ...
}

/* FactoryBeanRegistrySupport line 156 */
private Object doGetObjectFromFactoryBean(final FactoryBean<?> factory, final String beanName)
            throws BeanCreationException {

    ...
    // Key method!
    object = factory.getObject();
    ...
    return object;
}
```

看到`object = factory.getObject();`这个关键方法了么！  


再来说下`beforeSingletonCreation()`和`afterSingletonCreation()`这两个方法：  

``` java
/** Names of beans that are currently in creation. */
private final Set<String> singletonsCurrentlyInCreation =
        Collections.newSetFromMap(new ConcurrentHashMap<>(16));

/* DefaultSingletonBeanRegistry line 337 */
protected void beforeSingletonCreation(String beanName) {
    if (!this.inCreationCheckExclusions.contains(beanName) && !this.singletonsCurrentlyInCreation.add(beanName)) {
        throw new BeanCurrentlyInCreationException(beanName);
    }
}

/* DefaultSingletonBeanRegistry line 349 */
protected void afterSingletonCreation(String beanName) {
    if (!this.inCreationCheckExclusions.contains(beanName) && !this.singletonsCurrentlyInCreation.remove(beanName)) {
        throw new IllegalStateException("Singleton '" + beanName + "' isn't currently in creation");
    }
}
```

看到它们对`singletonsCurrentlyInCreation`这个`Map`的增删操作了吧。  


#### 4、getMergedLocalBeanDefinition(beanName)  

还记得我们上一章 [解析并注册BeanDefinition](http://yaoyichen.cn/spring/2020/04/11/spring-parse-and-register-bean-definition.html){:target="_blank"} 么？  

这里就要用到它了！  

``` java
/* AbstractBeanFactory line 1272 */
protected RootBeanDefinition getMergedLocalBeanDefinition(String beanName) throws BeansException {
    // Quick check on the concurrent map first, with minimal locking.
    RootBeanDefinition mbd = this.mergedBeanDefinitions.get(beanName);
    if (mbd != null && !mbd.stale) {
        return mbd;
    }
    return getMergedBeanDefinition(beanName, getBeanDefinition(beanName));
}
```


#### 5、getSingleton(beanName, ObjectFactory<?>)  

这是个`重载方法`，如果之前你无法从缓存获取`bean对象`，就要自己`create`了。  


``` java
/* DefaultSingletonBeanRegistry line 349 */
public Object getSingleton(String beanName, ObjectFactory<?> singletonFactory) {
    synchronized (this.singletonObjects) {
        Object singletonObject = this.singletonObjects.get(beanName);
        if (singletonObject == null) {
            ...
            beforeSingletonCreation(beanName);
            ...
            try {
                singletonObject = singletonFactory.getObject();
                newSingleton = true;
            }
            ...
            finally {
                ...
                afterSingletonCreation(beanName);
            }
            if (newSingleton) {
                addSingleton(beanName, singletonObject);
            }
        }
        return singletonObject;
    }
}
```


## createBean()  

我们来到第二个核心方法：`createBean()`。  

老规矩，看来看下这个方法中的核心内容：    


``` java
/* AbstractBeanFactory line 323 */
...
if (mbd.isSingleton()) {
    sharedInstance = getSingleton(beanName, () -> {
        try {
            return createBean(beanName, mbd, args);
        }
        ...
    });
    bean = getObjectForBeanInstance(sharedInstance, name, beanName, mbd);
}

/* AbstractAutowireCapableBeanFactory line 478 */
@Override
protected Object createBean(String beanName, RootBeanDefinition mbd, @Nullable Object[] args)
        throws BeanCreationException {
    ...
    RootBeanDefinition mbdToUse = mbd;

    /**
     * beanName      = car
     * resolvedClass = class com.frankie.demo.utils.CarFactoryBean
     */
    Class<?> resolvedClass = resolveBeanClass(mbd, beanName);
    if (resolvedClass != null && !mbd.hasBeanClass() && mbd.getBeanClassName() != null) {
        mbdToUse = new RootBeanDefinition(mbd);
        mbdToUse.setBeanClass(resolvedClass);
    }

    // Prepare method overrides.
    try {
        mbdToUse.prepareMethodOverrides();
    }
    ...
    try {
        Object beanInstance = doCreateBean(beanName, mbdToUse, args);
        if (logger.isTraceEnabled()) {
            logger.trace("Finished creating instance of bean '" + beanName + "'");
        }
        return beanInstance;
    }
    ...
}
```

真正干活的方法一般都是`do`开头的，所以`doCreateBean()`这个方法是关键中的关键！  

它主要做了以下四件事：  

- `createBeanInstance(beanName, mbd, args)`  
- `addSingletonFactory(beanName, ObjectFactory<?>)`  
- `populateBean(beanName, mbd, instanceWrapper)`  
- `initializeBean(beanName, exposedObject, mbd)`  


``` java
/* AbstractAutowireCapableBeanFactory line 548 */
protected Object doCreateBean(final String beanName, final RootBeanDefinition mbd, final @Nullable Object[] args)
            throws BeanCreationException {
    ...
    if (instanceWrapper == null) {
        instanceWrapper = createBeanInstance(beanName, mbd, args);
    }

    // Eagerly cache singletons to be able to resolve circular references
    // even when triggered by lifecycle interfaces like BeanFactoryAware.
    boolean earlySingletonExposure = (mbd.isSingleton() && this.allowCircularReferences &&
            isSingletonCurrentlyInCreation(beanName));
    if (earlySingletonExposure) {
        ...
        addSingletonFactory(beanName, () -> getEarlyBeanReference(beanName, mbd, bean));
    }

    // Initialize the bean instance.
    Object exposedObject = bean;
    try {
        populateBean(beanName, mbd, instanceWrapper);
        exposedObject = initializeBean(beanName, exposedObject, mbd);
    }
    ...
    return exposedObject;
}
```

#### 1、createBeanInstance(beanName, mbd, args)  

这个方法的作用是什么？  

它的作用就是为指定的`Bean`创建一个实例，一般由三种创建方式：  
`factory method`、 `constructor autowiring`和`simple instantiation`  

``` java
/**
 * AbstractAutowireCapableBeanFactory line 1162
 * Create a new instance for the specified bean, using an appropriate instantiation strategy:
 * factory method, constructor autowiring, or simple instantiation.
 */
protected BeanWrapper createBeanInstance(String beanName, RootBeanDefinition mbd, @Nullable Object[] args) {
    // Make sure bean class is actually resolved at this point.
    Class<?> beanClass = resolveBeanClass(mbd, beanName);

    // Shortcut when re-creating the same bean...
    boolean resolved = false;
    boolean autowireNecessary = false;
    if (args == null) {
        synchronized (mbd.constructorArgumentLock) {
            if (mbd.resolvedConstructorOrFactoryMethod != null) {
                resolved = true;
                autowireNecessary = mbd.constructorArgumentsResolved;
            }
        }
    }
    if (resolved) {
        if (autowireNecessary) {
            return autowireConstructor(beanName, mbd, null, null);
        }
        else {
            return instantiateBean(beanName, mbd);
        }
    }

    // Candidate constructors for autowiring?
    Constructor<?>[] ctors = determineConstructorsFromBeanPostProcessors(beanClass, beanName);
    if (ctors != null || mbd.getResolvedAutowireMode() == AUTOWIRE_CONSTRUCTOR ||
            mbd.hasConstructorArgumentValues() || !ObjectUtils.isEmpty(args)) {
        return autowireConstructor(beanName, mbd, ctors, args);
    }

    // Preferred constructors for default construction?
    ctors = mbd.getPreferredConstructors();
    if (ctors != null) {
        return autowireConstructor(beanName, mbd, ctors, null);
    }

    // No special handling: simply use no-arg constructor.
    return instantiateBean(beanName, mbd);
}
```

我们来看下`instantiateBean(beanName, mbd)`这个方法：  

``` java
/* AbstractAutowireCapableBeanFactory line 1302 */
protected BeanWrapper instantiateBean(final String beanName, final RootBeanDefinition mbd) {
    ...
    /**
     * beanName     = car
     * beanInstance = CarFactoryBean@4668
     */
    beanInstance = getInstantiationStrategy().instantiate(mbd, beanName, parent);
    BeanWrapper bw = new BeanWrapperImpl(beanInstance);
    initBeanWrapper(bw);
    return bw;
    ...
}
```

关键就在于`getInstantiationStrategy().instantiate(mbd, beanName, parent);`这个方法！  



#### 2、addSingletonFactory(beanName, ObjectFactory<?>)  


``` java
boolean earlySingletonExposure = (mbd.isSingleton() && this.allowCircularReferences &&
                isSingletonCurrentlyInCreation(beanName));
if (earlySingletonExposure) {
    if (logger.isTraceEnabled()) {
        logger.trace("Eagerly caching bean '" + beanName +
                "' to allow for resolving potential circular references");
    }
    addSingletonFactory(beanName, () -> getEarlyBeanReference(beanName, mbd, bean));
}
```

首先来看下`earlySingletonExposure`这个布尔变量，它什么时候才会是`true`？  

需要同时满足以下三个条件：  

- `Bean`为单例  
- 允许循环引用  
- 这个`Bean`是否正在构建  

让我们继续来看下`addSingletonFactory()`这个方法到底干了什么？  

``` java
/* DefaultSingletonBeanRegistry line 150 */
/** Cache of singleton objects: bean name to bean instance. */
private final Map<String, Object> singletonObjects = new ConcurrentHashMap<>(256);

/** Cache of singleton factories: bean name to ObjectFactory. */
private final Map<String, ObjectFactory<?>> singletonFactories = new HashMap<>(16);

/** Cache of early singleton objects: bean name to bean instance. */
private final Map<String, Object> earlySingletonObjects = new HashMap<>(16);

/** Set of registered singletons, containing the bean names in registration order. */
private final Set<String> registeredSingletons = new LinkedHashSet<>(256);

protected void addSingletonFactory(String beanName, ObjectFactory<?> singletonFactory) {
    Assert.notNull(singletonFactory, "Singleton factory must not be null");
    synchronized (this.singletonObjects) {
        if (!this.singletonObjects.containsKey(beanName)) {
            this.singletonFactories.put(beanName, singletonFactory);
            this.earlySingletonObjects.remove(beanName);
            this.registeredSingletons.add(beanName);
        }
    }
}
```

关键就在于往`singletonFactories`中添加`singletonFactory`。  

#### 3、populateBean(beanName, mbd, instanceWrapper)  

这个方法的作用很明显，就是将`BeanDefinition`中解析的`Property`赋给对象中的成员。  


``` java
/**
 * AbstractAutowireCapableBeanFactory line 1369
 * Populate the bean instance in the given BeanWrapper with the property values
 * from the bean definition.
 */
@SuppressWarnings("deprecation")  // for postProcessPropertyValues
protected void populateBean(String beanName, RootBeanDefinition mbd, @Nullable BeanWrapper bw) {

    PropertyValues pvs = (mbd.hasPropertyValues() ? mbd.getPropertyValues() : null);

    int resolvedAutowireMode = mbd.getResolvedAutowireMode();
    if (resolvedAutowireMode == AUTOWIRE_BY_NAME || resolvedAutowireMode == AUTOWIRE_BY_TYPE) {
        MutablePropertyValues newPvs = new MutablePropertyValues(pvs);
        // Add property values based on autowire by name if applicable.
        if (resolvedAutowireMode == AUTOWIRE_BY_NAME) {
            autowireByName(beanName, mbd, bw, newPvs);
        }
        // Add property values based on autowire by type if applicable.
        if (resolvedAutowireMode == AUTOWIRE_BY_TYPE) {
            autowireByType(beanName, mbd, bw, newPvs);
        }
        pvs = newPvs;
    }
    ...
    if (pvs != null) {
        applyPropertyValues(beanName, mbd, bw, pvs);
    }
}

/**
 * AbstractAutowireCapableBeanFactory line 1645
 */
protected void applyPropertyValues(String beanName, BeanDefinition mbd, BeanWrapper bw, PropertyValues pvs) {
    ...
    // Create a deep copy, resolving any references for values.
    List<PropertyValue> deepCopy = new ArrayList<>(original.size());
    ...
    pv.setConvertedValue(convertedValue);
    ...
    deepCopy.add(...)
    // Set our (possibly massaged) deep copy.
    try {
        bw.setPropertyValues(new MutablePropertyValues(deepCopy));
    }
    ...
}
```

让我们来看下这个`deepCopy`：  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/populateBean-deepCopy.png)  

#### 4、initializeBean(beanName, exposedObject, mbd)  

这个方法有什么作用呢？  


最常见的用法就是执行初始化方法。  

``` java
protected Object initializeBean(final String beanName, final Object bean, @Nullable RootBeanDefinition mbd) {
    ...
    try {
        invokeInitMethods(beanName, wrappedBean, mbd);
    }
    ...
}
```


## 循环依赖  

最后来讲下`Spring`中的循环依赖，主要分为以下三种：  

- 构造器循环依赖  
- `Setter`循环依赖  
- `Prototype`循环依赖  

#### 构造器循环依赖  

`Spring`中不允许构造器循环依赖：  


``` java
@Setter
public class ClassA {

    public ClassB b;

    public ClassA(){}

    public ClassA(ClassB b){
        this.b = b;
    }

    public void methodA(){
        b.methodB();
        System.out.println("Execute methodA().");
    }
}

@Setter
public class ClassB {

    public ClassC c;

    public ClassB(){}

    public ClassB(ClassC c){
        this.c = c;
    }

    public void methodB(){
        c.methodC();
        System.out.println("Execute methodB().");
    }
}

@Setter
public class ClassC {

    public ClassA a;

    public ClassC(){}

    public ClassC(ClassA a){
        this.a = a;
    }

    public void methodC(){
        a.methodA();
        System.out.println("Execute methodC().");
    }
}

<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="classA" class="com.frankie.demo.circleDependency.ClassA">
        <constructor-arg name="b" ref="classB"/>
    </bean>

    <bean id="classB" class="com.frankie.demo.circleDependency.ClassB">
        <constructor-arg name="c" ref="classC"/>
    </bean>

    <bean id="classC" class="com.frankie.demo.circleDependency.ClassC">
        <constructor-arg name="a" ref="classA"/>
    </bean>
</beans>

@Test
void constructorCDTest(){
    new ClassPathXmlApplicationContext("ConstructorCD.xml");
    // Caused by: org.springframework.beans.factory.BeanCurrentlyInCreationException:
    // Error creating bean with name 'classA': Requested bean is currently in creation:
    // Is there an unresolvable circular reference?
}
```

#### Setter循环依赖  

`Spring`中默认允许`Setter`循环依赖：  

``` java
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="classA" class="com.frankie.demo.circleDependency.ClassA">
        <property name="b" ref="classB"/>
    </bean>

    <bean id="classB" class="com.frankie.demo.circleDependency.ClassB">
        <property name="c" ref="classC"/>
    </bean>

    <bean id="classC" class="com.frankie.demo.circleDependency.ClassC">
        <property name="a" ref="classA"/>
    </bean>
</beans>

@Test
void setterCDTest(){
    ClassPathXmlApplicationContext ac = new ClassPathXmlApplicationContext("SetterCD.xml");
    ClassA classA = (ClassA) ac.getBean("classA");
    classA.methodA();
}
```

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/spring-setter-circle-dependency.png)  


如上图所示，这会造成死循环。  

那么如何禁止循环依赖？  

``` java
@Test
void setterCDTest(){
    ClassPathXmlApplicationContext ac = new ClassPathXmlApplicationContext("SetterCD.xml");
    ac.setAllowCircularReferences(false);
    ac.refresh();
    ClassA classA = (ClassA) ac.getBean("classA");
    classA.methodA();
    // Caused by: org.springframework.beans.factory.BeanCurrentlyInCreationException:
    // Error creating bean with name 'classA': Requested bean is currently in creation:
    // Is there an unresolvable circular reference?
}
```

#### Prototype循环依赖  

`Spring`也不允许`Prototype`循环依赖：  


``` java
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="classA" class="com.frankie.demo.circleDependency.ClassA" scope="prototype">
        <property name="b" ref="classB"/>
    </bean>

    <bean id="classB" class="com.frankie.demo.circleDependency.ClassB" scope="prototype">
        <property name="c" ref="classC"/>
    </bean>

    <bean id="classC" class="com.frankie.demo.circleDependency.ClassC" scope="prototype">
        <property name="a" ref="classA"/>
    </bean>
</beans>

@Test
void prototypeCDTest(){
    ApplicationContext ac = new ClassPathXmlApplicationContext("PrototypeCD.xml");
    ClassA classA = (ClassA) ac.getBean("classA");
    classA.methodA();
    // Caused by: org.springframework.beans.factory.BeanCurrentlyInCreationException:
    // Error creating bean with name 'classA': Requested bean is currently in creation:
    // Is there an unresolvable circular reference?
}
```

## Reference  
- `《Spring源码深度解析》 - 第五章 [Bean的加载]`  