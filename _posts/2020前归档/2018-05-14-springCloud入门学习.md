---
layout:     post
title:      SpringCloud入门学习（服务）
date:       2018-05-14
author:     yunzhs
header-img: img/Fate of Princess.jpg
catalog: true
tags:
    - SpringCloud
typora-root-url: ..
typora-copy-images-to: ..\img\posts
---

## 一、SpringCloud简介

spring cloud 为开发人员提供了快速构建分布式系统的一些工具，包括配置管理、服务发现、断路器、路由、微代理、事件总线、全局锁、决策竞选、分布式会话等等。它运行环境简单，可以在开发人员的电脑上跑。另外说明spring cloud是基于springboot的，所以需要开发中对springboot有一定的了解。

## 二、创建服务注册中心

在这里，我们需要用的的组件上Spring Cloud Netflix的Eureka ,eureka是一个服务注册和发现模块。

### **2.1 首先创建一个maven主工程。**

### **2.2 然后创建2个model工程:**

一个model工程作为服务注册中心，即Eureka Server,另一个作为Eureka Client。

下面以创建server为例子，详细说明创建过程：

右键工程->创建model-> 选择spring initialir 如下图：

![Snipaste_2018-05-14_14-32-17](/img/posts/Snipaste_2018-05-14_14-32-17.png)

![Snipaste_2018-05-14_14-38-52](/img/posts/Snipaste_2018-05-14_14-38-52.png)

下一步->选择cloud discovery->eureka server ,然后一直下一步就行了 ，然后就等加载依赖

![Snipaste_2018-05-14_14-40-00](/img/posts/Snipaste_2018-05-14_14-40-00.png)

### **2.3 启动一个服务注册中心**

只需要一个注解@EnableEurekaServer，这个注解需要在springboot工程的启动application类上加： 

![Snipaste_2018-05-14_14-52-22](/img/posts/Snipaste_2018-05-14_14-52-22.png)

### **2.4 **eureka是一个高可用的组件 

它没有后端缓存，每一个实例注册之后需要向注册中心发送心跳（因此可以在内存中完成），在默认情况下erureka server也是一个eureka client ,必须要指定一个 server。eureka server的配置文件appication.yml： 

```
server:
  port: 8761

eureka:
  instance:
    hostname: localhost
  client:
    registerWithEureka: false
    fetchRegistry: false
    serviceUrl:
      defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/
```

我是把properties删了再创建了一个yml的

通过eureka.client.registerWithEureka：false和fetchRegistry：false来表明自己是一个eureka server. 

### **2.5** eureka server 是有界面的

启动工程,打开浏览器访问：  [http://localhost:8761](http://localhost:8761/) ,界面如下： 

![Snipaste_2018-05-14_15-27-20](/img/posts/Snipaste_2018-05-14_15-27-20.png)

 不过是可以看见目前是没有服务可用，因为当前没有服务注册自然就没有服务

## 三、创建一个服务提供者

当client向server注册时，它会提供一些元数据，例如主机和端口，URL，主页等。Eureka server 从每个client实例接收心跳消息。 如果心跳超时，则通常将该实例从注册server中删除。

### 1.创建过程同server类似

只是要选择discover，和加一个web模块

### 2.在application中进行定义

```
@SpringBootApplication
@EnableEurekaClient
@RestController
public class ServiceHiApplication {

    public static void main(String[] args) {
        SpringApplication.run(ServiceHiApplication.class, args);
    }

    @Value("${server.port}")
    String port;
    @RequestMapping("/hi")
    public String home(@RequestParam String name) {
        return "hi "+name+",i am from port:" +port;
    }

}
```

### 3.application.yml配置文件如下： 

```
eureka:
  client:
    serviceUrl:
      defaultZone: http://localhost:8761/eureka/
server:
  port: 8762
spring:
  application:
    name: service-hi
```

需要指明spring.application.name,这个很重要，这在以后的服务与服务之间相互调用一般都是根据这个name 。  

### 4.效果

启动工程，打开[http://localhost:8761](http://localhost:8761/) ，即eureka server 的网址： 

![Snipaste_2018-05-14_15-42-04](/img/posts/Snipaste_2018-05-14_15-42-04.png)

这时打开 <http://localhost:8762/hi?name=yunzhs> ，你会在浏览器上看到 : 

` hi yunzhs,i am from port:8762 `

## 四、服务消费者

Spring cloud有两种服务调用方式，一种是ribbon+restTemplate，另一种是feign。

### 1.ribbon简介

**ribbon**是一个负载均衡客户端，可以很好的控制htt和tcp的一些行为。**Feign**默认集成了ribbon。 

ribbon是用来做一些负载均衡操作的，restTemplate 用来处理相关的需要处理的方法。

### 2.**Feign**简介

Feign是一个声明式的伪Http客户端，它使得写Http客户端变得更简单。使用Feign，只需要创建一个接口并注解。它具有可插拔的注解特性，可使用Feign 注解和JAX-RS注解。Feign支持可插拔的编码器和解码器。Feign默认集成了Ribbon，并和Eureka结合，默认实现了负载均衡的效果。 

```
Feign 采用的是基于接口的注解
Feign 整合了ribbon
```

### 3.Feign的配置

```
eureka:
  client:
    serviceUrl:
      defaultZone: http://localhost:8761/eureka/
server:
  port: 8765
spring:
  application:
    name: service-feign #自定义在eureka中的服务名
```

在引导类中开启eureka和feign的注解

>@EnableDiscoveryClient 
>
>@EnableFeignClients 

定义一个feign接口，通过@ FeignClient（“服务名”），来指定调用哪个服务。比如在代码中调用了service-hi服务的“/hi”接口，代码如下： 

```
@FeignClient(value = "service-hi")
public interface SchedualServiceHi {
    @RequestMapping(value = "/hi",method = RequestMethod.GET)
    String sayHiFromClientOne(@RequestParam(value = "name") String name);
}
```

在Web层的controller层，对外暴露一个”/hi”的API接口，通过上面定义的Feign客户端SchedualServiceHi 来消费服务。代码如下： 

```
@RestController
public class HiController {

    @Autowired
    SchedualServiceHi schedualServiceHi;
    @RequestMapping(value = "/hi",method = RequestMethod.GET)
    public String sayHi(@RequestParam String name){
        return schedualServiceHi.sayHiFromClientOne(name);
    }
}
```

启动程序，多次访问<http://localhost:8765/hi?name=forezp>,浏览器交替显示：

> hi forezp,i am from port:8762
>
> hi forezp,i am from port:8763





