---
layout: post
title: Spring依赖注入浅析
category: java
tags: [spring, java]
excerpt: Spring DI
---
## 概念理解
> 1.依赖注入  

- 谁注入谁？ `依赖对象`注入`IoC容器`。  

> 2.控制反转  

- 谁控制谁？控制什么？ IoC容器控制对象，`控制依赖对象的创建与注入`。
- 为什么称为反转？创建、注入对象的控制权由程序员的主观意愿转为IoC容器统一管理。
 
<br>
## 传统方式构建对象间依赖关系
```java
public class DvdPlayMissionImpossible {

    private MissionImpossibleCd missionImpossibleCd;

    public DvdPlayMissionImpossible(){
        this.missionImpossibleCd = new MissionImpossibleCd(); // Tight coupling
    }

    public void play(){
        System.out.println("一台只能看" + missionImpossibleCd.getCompactDiskName() + "的DVD");
    }
}

public class DvdPlayFurious {

    private FuriousCd furiousCd;

    public DvdPlayFurious(){
        this.furiousCd = new FuriousCd(); // Tight coupling
    }

    public void play(){
        System.out.println("一台只能看" + furiousCD.getCompactDiskName() + "的DVD");
    }
}


public class MissionImpossibleCd {

    public String getCompactDiskName(){
        return "碟中谍";
    }
}

public class FuriousCd {

    public String getCompactDiskName(){
        return "速度与激情";
    }
}

// Traditional Way.
@Test
public void dvdPlayerMissionImpossibleTest(){
    DvdPlayMissionImpossible dvdPlayMissionImpossible = new DvdPlayMissionImpossible();
    DvdPlayFurious           dvdPlayFurious           = new DvdPlayFurious();
    dvdPlayMissionImpossible.play(); // 一台只能看碟中谍的DVD
    dvdPlayFurious.play();           // 一台只能看速度与激情的DVD
}
```
 
<br>
## DI方式构建对象间依赖关系  

>所有CD都拥有电影名称，因此抽象成一个CompactDisk接口。  

```java
public interface CompactDisk {
    String getCDName();
}

@Component
public class MissionImpossible implements CompactDisk {

    @Override
    public String getCDName() {
        return "碟中谍";
    }
}

@Component
public class Furious implements CompactDisk{

    @Override
    public String getCDName() {
        return "速度与激情";
    }
}
```
  
<br>

>所有DVD播放器都拥有播放的功能，因此抽象成一个MediaPlayer接口。  

```java
public interface MediaPlayer {
    void play();
}

@Component
public class DvdPlayer implements MediaPlayer {

    @Autowired
    @Qualifier("missionImpossible")
    private CompactDisk cd;

    @Override
    public void play() {
        System.out.println("一台可以看" + cd.getCDName() + "的DVD");
    }
}

@Component
public class VCDPlayer implements MediaPlayer {

    @Autowired
    @Qualifier("furious")
    private CompactDisk cd;

    @Override
    public void play() {
        System.out.println("一台可以看" + cd.getCDName() + "的VCD");
    }
}
```
 
<br>
>单元测试  

```java
@Autowired
DvdPlayer dvdPlayer;

@Autowired
VCDPlayer vcdPlayer;

// DI Way.
@Test
public void dvdAndVcdPlayerTest(){
    dvdPlayer.play(); // 一台可以看碟中谍的DVD
    vcdPlayer.play(); // 一台可以看速度与激情的VCD
}
```
 
<br>
## 依赖注入的优势
- 对象之间解耦，毕竟一台只能看一部电影的播放器，我想没人愿意买吧。
- 同一个对象，IoC容器只需要创建一次。
- IoC容器会帮你自动匹配对象之间复杂的依赖关系。
 
<br>
## 参考资料
- [IoC 之 2.1 IoC基础 ——跟我学Spring3](https://www.iteye.com/blog/jinnianshilongnian-1413846){:target="_blank"}
- [Spring IoC有什么好处呢？](https://www.zhihu.com/question/23277575/answer/169698662){:target="_blank"}
- 《Spring实战》 第二章