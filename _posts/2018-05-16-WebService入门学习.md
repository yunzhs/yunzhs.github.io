---
layout:     post
title:      WebService入门学习
date:       2018-05-15
author:     yunzhs
header-img: img/Fate of Princess.jpg
catalog: true
tags:
    - WebService
typora-root-url: ..
typora-copy-images-to: ..\img\posts
---

# 一.WebService简介

## 1.什么是WebService?

通过使用WebService，您的应用程序可以向全世界发布信息，或提供某项功能，它是基于Web的服务，通过Web进行发布、查找和使用。

WebService脚本平台需支持XML+HTTP。

HTTP协议是最常用的因特网协议。

XML提供了一种可用于不同的平台和编程语言之间的语言。

## 2.为什么要使用webservice?

最重要的事情是协同工作。

1.跨平台调用（WebService不局限于操作系统，你可以在Windows上调用linux上的WebService服务，反之亦然，其他系统同理）；

2.跨语言调用（WebService不局限于编程语言，你可以在Java语言中调用C#语言提供的WebService服务，反之亦然，其他语言同理）；

3.可远程调用（通过使用WebService，您的应用程序可以向全世界发布信息，或提供某项功能，只要有Internet）。

# 二.使用idea开发WebService

### 1.首先新建一个项目,webservice

![1526452999201](/img/posts/1526452999201.png)

### 2.表结构和示例代码:

![1526453129689](/img/posts/1526453129689.png)

### 3.直接启动,即可运行

![1526453185106](/img/posts/1526453185106.png)

### 4.自己写一个server,发现也可以运行

![1526453805801](/img/posts/1526453805801.png)

### 5.创建客户端

![1534749626382](/img/posts/1534749626382.png)

输入客户端项目名，finish即可，项目创建成功会自动跳出如下界面，手动可以右键项目–>webService–>Generate Java Code From Wsdl 即可 

![1534749667056](/img/posts/1534749667056.png)

```
//随便从项目中找的一段测试类
SaleTicketServiceServiceLocator locator = new SaleTicketServiceServiceLocator();
SaleTicketService service = locator.getSaleTicketServicePort();
String lineInfo = service.getPlanInfo("1");
System.out.println(lineInfo);
```

然后客服端会自动给我们生产一段测试代码,接下来我们只需要将服务端用tomcat启动,就能把它作为一个完整的项目来进行开发了.

