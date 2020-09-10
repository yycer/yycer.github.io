---
layout: post
title: Java - System类
category: java-basic
tags: [java]
excerpt: Java System Class
---

本文介绍下`Java`中系统类`System`的常见用法。   

需要注意`System`类被`final`修饰，所以不能被其他类继承，   

同时它只有一个被`private`修饰的构造函数，所以不能构造实例。  


## in & out & err  

分别代表标准输入、输出、错误流，举个例子：    

``` java
var br = new BufferedReader(new InputStreamReader(System.in));
String inputLine = br.readLine();

var sb = new StringBuilder(inputLine);
sb.reverse();

System.out.println("Input    string: " + inputLine);
System.out.println("Reversed string: " + sb);
System.err.println("Reversed string: " + sb);

br.close();

/** ------------------------*/
// Hello World.
// Input    string: Hello World.
// Reversed string: .dlroW olleH
// Reversed string: .dlroW olleH
```


## Array Copy  

我们还可以处理数组之间的复制：  

``` java
public static native void arraycopy(Object src,  int  srcPos,
                                    Object dest, int destPos,
                                    int length);
```

``` java
final int N = 10;
int[] src = new int[N];
for (int i = 0; i < N; i++){
    src[i] = i;
}
System.out.println("src = " + Arrays.toString(src));

int[] des = new int[N >>> 1];
System.arraycopy(src, 3, des, 0, des.length);
System.out.println("des = " + Arrays.toString(des));

/** ------------------------*/
// src = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
// des = [3, 4, 5, 6, 7]
```


## System Properties  

还可以查询和修改系统属性：  

``` java
Properties properties = System.getProperties();
for (Object key: properties.keySet()){
    System.out.println("key = " + key + ", val = "+ properties.getProperty((String) key));
}

/** ------------------------*/
// key = file.separator, val = \
// key = java.class.path, val = D:\Playground\improve-java\target\classes;...
// key = java.home, val = C:\Java\jdk-11.0.2
// key = java.vendor, val = Oracle Corporation
// key = java.vendor.url, val = http://java.oracle.com/
// key = java.version, val = 11.0.2
// key = line.separator, val = 
// key = os.arch, val = amd64
// key = os.name, val = Windows 10
// key = os.version, val = 10.0
// key = path.separator, val = ;
// key = user.dir, val = D:\Playground\improve-java
// key = user.home, val = C:\Users\...
// key = user.name, val = ...
// key = java.runtime.version, val = 11.0.2+9-LTS
// key = java.runtime.name, val = Java(TM) SE Runtime Environment
// key = file.encoding, val = UTF-8
// key = java.class.version, val = 55.0
// ...
```


## Environmental Variables  

同时，还可以通过`System`类查询环境变量：  



``` java
Map<String, String> env = System.getenv();
for (var key: env.keySet()){
   System.out.println("key = " + key + ", val = " + env.get(key));
}

/** ------------------------*/
// key = JAVA_8_HOME, val = C:\Java8\jdk1.8.0_201\bin
// key = JAVA_11_HOME, val = C:\Java\jdk-11.0.2\bin
// key = JAVA_HOME, val = C:\Java\jdk-11.0.2\bin
// key = LANG, val = zh_CN
// key = SystemDrive, val = C:
// key = USERNAME, val = ...
// key = Path, val = ...
// ...
```


## Current Time & GC   


``` java
System.out.println(System.currentTimeMillis());
System.gc();
```


## Reference  
- [java.lang.System Example](https://examples.javacodegeeks.com/core-java/lang/system/java-lang-system-example/){:target="_blank"}  
- [System Properties](https://docs.oracle.com/javase/tutorial/essential/environment/sysprop.html){:target="_blank"}  
