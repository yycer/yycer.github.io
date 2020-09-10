---
layout: post
title: Java - 对象复制  
category: java-basic
tags: [java]
excerpt: Java Clone Object  
---

本文梳理一下`Java`中的对象复制。   

我们可以发现在`Object`类中有如下方法：  

``` java
protected native Object clone() throws CloneNotSupportedException;
```

那么如何使用它来实现对象的复制呢？   


## Shallow & Deep Clone  

我们先准备一下我们需要的类：  

``` java
@Data
@AllArgsConstructor
public class Employee {

    private int id;
    private String name;
    private Department department;
}

@Data
@AllArgsConstructor
public class Department {

    private int id;
    private String name;
}
```

然后，让`Employee`实现`Cloneable`接口，并重写`clone()`方法。  

``` java
@Data
@AllArgsConstructor
public class Employee implements Cloneable{

    private int id;
    private String name;
    private Department department;

    @Override
    public Object clone() throws CloneNotSupportedException {
        return super.clone();
    }
}
```

接着，我们创建一个原始对象`original`和复制对象`cloned`，并修改`cloned`的部门名称：  


``` java
public class Test {
    public static void main(String[] args) throws CloneNotSupportedException {

        Department dept = new Department(1, "Technical");
        Employee original = new Employee( 24, "yyc", dept);

        var cloned = (Employee) original.clone();

        System.out.println("Cloned   id           : " + cloned.getId());
        System.out.println("Cloned name           : " + cloned.getName());
        System.out.println("Cloned department   id: " + cloned.getDepartment().getId());
        System.out.println("Cloned department name: " + cloned.getDepartment().getName());
        System.out.println("-------------------------------------");

        cloned.getDepartment().setName("Finance");
        System.out.println("Original department name: " + original.getDepartment().getName());
        System.out.println("Cloned   department name: " + cloned.getDepartment().getName());
    }
}

/** ------------------------------------*/
// Cloned   id           : 24
// Cloned name           : yyc
// Cloned department   id: 1
// Cloned department name: Technical
// -------------------------------------
// Original department name: Finance
// Cloned   department name: Finance
```

与我们的期望不一致，我们期望只将`cloned`的部门名称改成`Finance`，而`original`的部门名称仍为`Technical`。  

这就是`Shallow Copy`，浅复制。  

来看下示意图：  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/shallow_copy.png)  

那么，如何达到我们的期望？  

也就是如何做到`Deep Copy`？下面提供三种解决方案：  

- 所有相关对象均实现`Cloneable`接口  
- 复制构造器  
- 序列化： `common-lang3`、`Jackson`、`Gson`  

示意图如下：  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/deep_copy.png)  



## Cloneable  

注意，`Employee`和`Department`类都需要实现`Cloneable`接口，并重写`clone()`方法。  

``` java
@Data
@AllArgsConstructor
public class Employee implements Cloneable{

    private int id;
    private String name;
    private Department department;

    @Override
    public Object clone() throws CloneNotSupportedException {
        Employee e = (Employee) super.clone();
        e.setDepartment((Department) this.department.clone());
        return e;
    }
}

@Data
@AllArgsConstructor
public class Department implements Cloneable {

    private int id;
    private String name;

    @Override
    public Object clone() throws CloneNotSupportedException {
        return super.clone();
    }
}

public class Test {
    public static void main(String[] args) throws CloneNotSupportedException {

        Department   dept = new Department(1, "Technical");
        Employee original = new Employee( 24, "yyc", dept);

        var cloned = (Employee) original.clone();
        System.out.println("Cloned   id           : " + cloned.getId());
        System.out.println("Cloned name           : " + cloned.getName());
        System.out.println("Cloned department   id: " + cloned.getDepartment().getId());
        System.out.println("Cloned department name: " + cloned.getDepartment().getName());
        System.out.println("-------------------------------------");

        cloned.getDepartment().setName("Finance");
        System.out.println("Original department name: " + original.getDepartment().getName());
        System.out.println("Cloned   department name: " + cloned.getDepartment().getName());

    }
}

/** ------------------------------------*/
// Cloned   id           : 24
// Cloned name           : yyc
// Cloned department   id: 1
// Cloned department name: Technical
// -------------------------------------
// Original department name: Technical
// Cloned   department name: Finance
```


## Copy Constructor    

关键在于将原始对象作为参数传入该类的构造器，对于基本类型和字符串直接赋值，  

对于引用类型则需要创建一个新的对象。  

``` java
@Data
@AllArgsConstructor
public class Employee {

    private int id;
    private String name;
    private Department department;

    public Employee(Employee e){
        this(e.getId(), e.getName(), new Department(e.getDepartment()));
    }
}

@Data
@AllArgsConstructor
public class Department {

    private int id;
    private String name;

    public Department(Department d){
        this(d.getId(), d.getName());
    }
}

public class Test {
    public static void main(String[] args) throws CloneNotSupportedException {

        Department   dept = new Department(1, "Technical");
        Employee original = new Employee( 24, "yyc", dept);

        var cloned = new Employee(original);
        System.out.println("Cloned   id           : " + cloned.getId());
        System.out.println("Cloned name           : " + cloned.getName());
        System.out.println("Cloned department   id: " + cloned.getDepartment().getId());
        System.out.println("Cloned department name: " + cloned.getDepartment().getName());
        System.out.println("-------------------------------------");

        cloned.getDepartment().setName("Finance");
        System.out.println("Original department name: " + original.getDepartment().getName());
        System.out.println("Cloned   department name: " + cloned.getDepartment().getName());

    }
}

/** ------------------------------------*/
// Cloned   id           : 24
// Cloned name           : yyc
// Cloned department   id: 1
// Cloned department name: Technical
// -------------------------------------
// Original department name: Technical
// Cloned   department name: Finance
```


## common-lang3  

注意`Employee`和`Department`类都要实现`Serializable`接口。  

``` java
@Data
@AllArgsConstructor
public class Employee implements Serializable {

    private int id;
    private String name;
    private Department department;
}

@Data
@AllArgsConstructor
public class Department implements Serializable {

    private int id;
    private String name;
}

public class Test {
    public static void main(String[] args) {

        Department   dept = new Department(1, "Technical");
        Employee original = new Employee( 24, "yyc", dept);
        Employee cloned = SerializationUtils.clone(original);

        System.out.println("Cloned   id           : " + cloned.getId());
        System.out.println("Cloned name           : " + cloned.getName());
        System.out.println("Cloned department   id: " + cloned.getDepartment().getId());
        System.out.println("Cloned department name: " + cloned.getDepartment().getName());
        System.out.println("-------------------------------------");

        cloned.getDepartment().setName("Finance");
        System.out.println("Original department name: " + original.getDepartment().getName());
        System.out.println("Cloned   department name: " + cloned.getDepartment().getName());

    }
}
/** ------------------------------------*/
// Cloned   id           : 24
// Cloned name           : yyc
// Cloned department   id: 1
// Cloned department name: Technical
// -------------------------------------
// Original department name: Technical
// Cloned   department name: Finance
```


## Jackson  

注意`Employee`和`Department`类都要有一个默认构造函数。  

``` java
@Data
@AllArgsConstructor
public class Employee {

    private int id;
    private String name;
    private Department department;

    public Employee(){ }
}

@Data
@AllArgsConstructor
public class Department {

    private int id;
    private String name;

    public Department(){}
}

public class Test {
    public static void main(String[] args) throws JsonProcessingException {

        Department   dept = new Department(1, "Technical");
        Employee original = new Employee( 24, "yyc", dept);

        ObjectMapper om = new ObjectMapper();
        Employee cloned = om.readValue(om.writeValueAsString(original), Employee.class);

        System.out.println("Cloned   id           : " + cloned.getId());
        System.out.println("Cloned name           : " + cloned.getName());
        System.out.println("Cloned department   id: " + cloned.getDepartment().getId());
        System.out.println("Cloned department name: " + cloned.getDepartment().getName());
        System.out.println("-------------------------------------");

        cloned.getDepartment().setName("Finance");
        System.out.println("Original department name: " + original.getDepartment().getName());
        System.out.println("Cloned   department name: " + cloned.getDepartment().getName());

    }
}

/** ------------------------------------*/
// Cloned   id           : 24
// Cloned name           : yyc
// Cloned department   id: 1
// Cloned department name: Technical
// -------------------------------------
// Original department name: Technical
// Cloned   department name: Finance
```

## Gson  

对于`Model`不需要做任何修改！  

``` java
@Data
@AllArgsConstructor
public class Employee {

    private int id;
    private String name;
    private Department department;
}

@Data
@AllArgsConstructor
public class Department {

    private int id;
    private String name;
}

public class Test {
    public static void main(String[] args) {

        Department   dept = new Department(1, "Technical");
        Employee original = new Employee( 24, "yyc", dept);

        Gson gson = new Gson();
        Employee cloned = gson.fromJson(gson.toJson(original), Employee.class);
        System.out.println("Cloned   id           : " + cloned.getId());
        System.out.println("Cloned name           : " + cloned.getName());
        System.out.println("Cloned department   id: " + cloned.getDepartment().getId());
        System.out.println("Cloned department name: " + cloned.getDepartment().getName());
        System.out.println("-------------------------------------");

        cloned.getDepartment().setName("Finance");
        System.out.println("Original department name: " + original.getDepartment().getName());
        System.out.println("Cloned   department name: " + cloned.getDepartment().getName());

    }
}

/** ------------------------------------*/
// Cloned   id           : 24
// Cloned name           : yyc
// Cloned department   id: 1
// Cloned department name: Technical
// -------------------------------------
// Original department name: Technical
// Cloned   department name: Finance

// Process finished with exit code 0
```


## Reference  
- [How to Make a Deep Copy of an Object in Java](https://www.baeldung.com/java-deep-copy){:target="_blank"}  
- [Java clone – deep and shallow copy – copy constructors](https://howtodoinjava.com/java/cloning/a-guide-to-object-cloning-in-java/#comment-25762){:target="_blank"}  

