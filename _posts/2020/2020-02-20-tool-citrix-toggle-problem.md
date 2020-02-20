---
layout: post
title: 工具 - Critix toggle problem
category: tool
tags: [tool]
excerpt: Critix toggle problem
---

## citrix切换程序问题

由于疫情原因，终于开通了citrix远程办公，但是呢，遇到一个很难受的问题：  

在远程桌面切换程序时，会切换回本机，非常的不方便。  

找了好久，终于找到一个有效的解决方案：  

- Step1  

`Win + R -> regedit.exe -> Enter`，打开注册表编辑器。  


- Step2  

> HKEY_LOCAL_MACHINE\SOFTWARE\WOW6432Node\Citrix\ICA Client\Engine\Lockdown Profiles\All Regions\Lockdown\Virtual Channels\Keyboard\

在这个路径下，找到`TransparentKeyPassthrough`，将其值改为`Remote`  

起飞~  

## `Reference`
- [Enable toggle between applications using Alt-Tab keys within a remote desktop session](https://support.citrix.com/article/CTX232298){:target="_blank"}  

