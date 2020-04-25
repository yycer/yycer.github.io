---
layout: post
title: Java - [容器]Set
category: collection
tags: [spring, java]
excerpt: Java Collection Set
---

`Update 2020_0425`

本文梳理下`Java`容器中的`Set`，主要从以下四个角度：  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/collection_four_dimension.png)  


先来看下`HashSet`、`LinkedHashSet`、`TreeSet`和`Collection`的关系图：  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/set_framework.png)  

解释下这张图：  

- `蓝色实线`表示`类`之间的继承关系  
- `绿色实线`代表`接口`之间的继承关系  
- `绿色虚线`代表`类`实现了某个`接口`  


## `HashSet`  

``` java
@Getter
public class Student {

    private int id;
    private String name;

    public Student(int id, String name){
        this.id = id;
        this.name = name;
    }


    @Override
    public boolean equals(Object obj){
        if (this == obj) return true;
        if (obj == null || this.getClass() != obj.getClass()){
            return false;
        }
        Student stu = (Student) obj;
        boolean ret = this.getId() == stu.getId() &&
                this.getName().equals(stu.getName());

        System.out.println("The result of equals() is " + ret + ", by " + name);
        return ret;
    }

//    @Override
//    public boolean equals(Object obj){
//        boolean ret = super.equals(obj);
//        System.out.println("The result of equals() from Object is " + ret);
//        return ret;
//    }

    @Override
    public int hashCode(){
        final int prime = 31;
        int ret = 1;
        ret = prime * ret + id;
        ret = prime * ret + (("".equals(name) ? 0 : name.hashCode()));
        System.out.println("The result of hasCode is " + ret + ", by " + name);
        return ret;
    }

    @Override
    public String toString(){
        return String.format("Student[id=%d, name=%s]", id, name);
    }
}

@Test
void hashSetTest(){
    HashSet<Student> students = new HashSet<>();
    students.add(new Student(20, "yyc"));
    students.add(new Student(22, "Jack"));
    students.add(new Student(24, "Marion"));
    students.add(new Student(20, "yyc"));
    students.add(null);

    System.out.println(students);
    // The result of hasCode  is 121712,      by yyc
    // The result of hasCode  is 2302570,     by Jack
    // The result of hasCode  is -1997438797, by Marion
    // The result of hasCode  is 121712,      by yyc
    // The result of equals() is true,        by yyc
    // [null, Student[id=20, name=yyc], Student[id=24, name=Marion], Student[id=22, name=Jack]]
}
```

可以得出以下结论：  

- 是否允许添加`null`？ `是`  
- 是否允许添加重复元素？ `否`  
- 是否维持插入顺序？ `否` 
- 是否线程安全？ `否`  

非线程安全的原因是因为追溯`add()`方法并没有使用同步方法、并且操作的变量并没有使用`final`修饰。  

再讲个有意思的地方：`HashSet如何判断一个元素是否重复？`  

其实从上面的注释中可以看出端倪： `equals() == true，同时hashCode()值相等`。  



## `LinkedHashSet`  


``` java
@Test
void linkedHashSetTest(){
    LinkedHashSet<String> lhSet = new LinkedHashSet<>();
    lhSet.add("yyc");
    lhSet.add("yyc");
    lhSet.add("frankie");
    lhSet.add("Jack");
    lhSet.add("Marion");
    lhSet.add(null);
    System.out.println("Origin  Sequence: " + lhSet);
    lhSet.remove("yyc");
    lhSet.add("yyc");
    System.out.println("Changed Sequence: " + lhSet);

    // Origin  Sequence: [yyc, frankie, Jack, Marion, null]
    // Changed Sequence: [frankie, Jack, Marion, null, yyc]
}
```

可以得出以下结论：  

- 是否允许添加`null`？ `是`  
- 是否允许添加重复元素？ `否`  
- 是否维持插入顺序？ `是`
- 是否线程安全？ `否` 


## `TreeSet`  

``` java
@Test
void treeSetTest(){
    TreeSet<Integer> treeSet = new TreeSet<>();
    treeSet.add(5);
    treeSet.add(1);
    treeSet.add(3);
    treeSet.add(2);
    treeSet.add(2);
    // treeSet.add(null);
    /** [1, 2, 3, 5] */
    System.out.println(treeSet);
    /** (-∞, 3): [1, 2] */
    System.out.println("(-∞, 3): " + treeSet.headSet(3));
    /** [3, +∞): [3, 5] */
    System.out.println("[3, +∞): " + treeSet.tailSet(3));
    /** [-2, 3): [1, 2] */
    System.out.println("[-2, 3): " + treeSet.subSet(-2, 3));
}
```

可以得出以下结论：  

- 是否允许添加`null`？ `否`  
- 是否允许添加重复元素？ `否`  
- 是否维持插入顺序？ `否，根据元素大小升序排列`  
- 是否线程安全？ `否`  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/treeSet_cannot_add_null.png)  
    

最后总结如下：  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/set_features.png)  

## `Reference`  

- [Java集合总结 之 Set](https://lzyz.fun/javaset/comment-page-1/?unapproved=119&moderation-hash=13406ce5b7ca9fd35e5dfadfc4ce8951#comment-119){:target="_blank"}  
- `《疯狂Java讲义》 - 8.3 Set集合`  

