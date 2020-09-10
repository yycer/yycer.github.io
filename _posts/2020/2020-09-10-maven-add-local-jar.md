---
layout: post
title: Maven - 向项目中导入本地Jar包
category: tool
tags: [tool]
excerpt: Maven import local jar into project  
---

本文主要介绍一下如何将一个第三方`Jar`包导入项目中，以`Oracle Database Driver`为例，  
介绍两种方式，第一种使用`mvn install:install-file`命令，第二种使用`systemPath`。  

## 获取Jar包   

首先，进入`Oracle JDBC`下载官网： [jdbc-downloads](https://www.oracle.com/database/technologies/appdev/jdbc-downloads.html){:target="_blank"}  


然后，选择对应的版本：  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/oracle-db-drivers-19.7.png)  


接着，下载与你的`JDK`匹配的`Jar`包：  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/oracle-db-drivers-ojdbc8.png)  



## 命令法   

首先，我们先来看下`mvn install:install-file`命令的参数：  

``` java
mvn install:install-file 
-Dfile=<path-to-file> 
-DgroupId=<group-id> 
-DartifactId=<artifact-id> 
-Dversion=<version> 
-Dpackaging=<packaging>
```

举个例子：  

``` java
mvn install:install-file 
-Dfile=D:/Downloads/ojdbc8.jar
-DgroupId=com.oracle 
-DartifactId=ojdbc8 
-Dversion=19.7 
-Dpackaging=jar 
```

执行成功后，在本地仓库中就会看到这个`Jar`包：  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/oracle-db-drivers-local-repository.png)  

然后在`pom.xml`文件中添加如下依赖：  

``` java
<dependency>
    <groupId>com.oracle</groupId>
    <artifactId>ojdbc8</artifactId>
    <version>19.7</version>
</dependency>
```

此时我们就可以使用其中的类了：  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/oracle-db-drivers-usage.png)  


## 设置系统路径法  

直接在`pom.xml`文件中添加如下依赖即可：  

``` java
<dependency>
    <groupId>com.oracle</groupId>
    <artifactId>ojdbc</artifactId>
    <version>8</version>
    <scope>system</scope>
    <systemPath>D:/Downloads/ojdbc8.jar</systemPath>
</dependency>
```


## Reference  

- [How to add Oracle JDBC driver in your Maven local repository](https://mkyong.com/maven/how-to-add-oracle-jdbc-driver-in-your-maven-local-repository/){:target="_blank"}  
- [Guide to installing 3rd party JARs](https://maven.apache.org/guides/mini/guide-3rd-party-jars-local.html#guide-to-installing-3rd-party-jars){:target="_blank"}  
