---
layout:     post
title:      Spring Boot入门(整合一个ActiveMQ)
date:       2018-1-27
author:     yunzhs
header-img: img/Archer.jpg
catalog: true
tags:
    - SpringBoot
    - ActiveMQ
typora-root-url: ..
typora-copy-images-to: ..\img\posts
---

## 1.前言

​        Spring 的组件代码是轻量级的，但它的配置却是重量级的。一开始，Spring 用 XML 配置，而且是很多 XML 配置。Spring 2.5 引入了基于注解的组件扫描，这消除了大量针对应用程序自身组件的显式 XML 配置。Spring 3.0 引入了基于 Java 的配置，这是一种类型安全的可重构配置方式，可以代替 XML。所有这些配置都代表了开发时的损耗。因为在思考 Spring 特性配置和解决业务问题之间需要进行思维切换，所以写配置挤占了写应用程序逻辑的时间。和所有框架一样，Spring 实用，但与此同时它要求的回报也不少。

​	除此之外，项目的依赖管理也是件吃力不讨好的事情。决定项目里要用哪些库就已经够让人头痛的了，你还要知道这些库的哪个版本和其他库不会有冲突，这难题实在太棘手。依赖管理也是一件，添加依赖不是写应用程序代码。一旦选错了依赖的版本，那就有的搞了。



> Spring Boot 让这一切成为了过去。

​	

​	Spring Boot 是 Spring 社区较新的一个项目。该项目的目的是帮助开发者更容易的创建基于 Spring 的应用程序和服务，让更多人的人更快的对 Spring 进行入门体验，为 Spring 生态系统提供了一种固定的、约定优于配置风格的框架。

Spring Boot 具有如下特性：

（1）为基于 Spring 的开发提供更快的入门体验

（2）开箱即用，没有代码生成，也无需 XML 配置。同时也可以修改默认值来满足特定的需求。

（3）提供了一些大型项目中常见的非功能性特性，如嵌入式服务器、安全、指标，健康检测、外部配置等。

（4）Spring Boot 并不是不对 Spring 功能上的增强，而是提供了一种快速使用 Spring 的方式。

## 2.Spring入门Demo

### 创建Maven工程 springboot_demo（打包方式jar）

####1.在pom.xml中添加如下依赖 :

```
  <parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
	<version>1.4.0.RELEASE</version>
  </parent>  
  <dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
  </dependencies>
```

导入后,会发现spring工程中导入了很多jar包

#### 2.变更JDK版本

默认情况下工程的JDK版本是1.6,而我们常使用的是1.7,1.8,所以我们可以在pom.xml中添加一下配置:

```
  <properties>   
    <java.version>1.8</java.version>
  </properties>
```

#### 3.引导类的创建

```
package com.yunzhs.demo;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
@SpringBootApplication
public class Application {
	public static void main(String[] args) {
		SpringApplication.run(Application.class, args);
	}
}
```

解释一下:

@SpringBootApplication其实就是以下三个注解的总和

@Configuration： 用于定义一个配置类

@EnableAutoConfiguration ：Spring Boot会自动根据你jar包的依赖来自动配置项目。

@ComponentScan： 告诉Spring 哪个packages 的用注解标识的类 会被spring自动扫描并且装入bean容器。

我们直接执行这个引导类，会发现控制台出现的这个标识

**spring boot框架内嵌服务器,因此启动spring boot会直接启动一个tomcat服务器**

![1517705854841](/img/posts/1517705854841.png)

#### 4.写一个简单的hello world

直接随便的建一个controller类

```
@RestController
public class HelloWorldController {
	@RequestMapping("/info")
	public String info(){
		return "HelloWorld";		
	}		
}
```

访问http://localhost:8080/info,即可查看效果

#### 5.修改tomcat启动端口

在src/main/resources下创建application.properties

```
server.port=8088
```

我要在类中读取这个配置信息，修改HelloWorldController  

```
	@Autowired
	private Environment env;
	
	`````
	@RequestMapping("/info")
	public String info(){
		return "HelloWorld~~"+env.getProperty("url");
```

#### 6.实现简单的热部署 

两种方式:

1.在pom文件中直接依赖

```
	<dependency>  
	    <groupId>org.springframework.boot</groupId>  
	    <artifactId>spring-boot-devtools</artifactId>  
	</dependency>  
```
2.在pom文件中,添加一个插件

```
		<plugin>
          <groupId>org.springframework.boot</groupId>
          <artifactId>spring-boot-maven-plugin</artifactId>
          <dependencies>
            <!-- spring热部署 -->
            <dependency>
              <groupId>org.springframework</groupId>
              <artifactId>springloaded</artifactId>
              <version>1.2.6.RELEASE</version>
            </dependency>
          </dependencies>
          <configuration>
            <mainClass>cn.springboot.Mainspringboot</mainClass>
          </configuration>
        </plugin>
```



真正实现热部署的只是后者，前者只是实现了热启动而已，从控制台日志就可以看出来。

当然,springboot自带的热部署,只能实现一些后台java代码热部署的修改,如果配置文件发生改变.就不能实现热部署了,因此,我们如果真正想用热部署,还是要用**JRebel**

#### 7.Spring Boot与ActiveMQ整合

**1.使用内嵌服务**

（1）在pom.xml中引入ActiveMQ起步依赖

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-activemq</artifactId>
</dependency>
```

（2）创建消息生产者

```
/**
 * 消息生产者
 * @author Administrator
 */
@RestController
public class QueueController {
	@Autowired
	private JmsMessagingTemplate jmsMessagingTemplate;

	@RequestMapping("/send")
	public void send(String text){
		jmsMessagingTemplate.convertAndSend("yunzhs", text);
	}
}
```

（3）创建消息消费者

```
@Component
public class Consumer {
	@JmsListener(destination="yunzhs")
	public void readMessage(String text){
		System.out.println("接收到消息："+text);
	}	
}
```

测试：启动服务后，在浏览器执行 

[http://localhost:8088/send](http://localhost:8080/send).do?text=aaaaa

即可看到控制台输出消息提示。Spring Boot内置了ActiveMQ的服务，所以我们不用单独启动也可以执行应用程序。

**2.使用外部服务**

在src/main/resources下的application.properties增加配置, 指定ActiveMQ的地址

```
spring.activemq.broker-url=tcp://服务器地址:61616
```

运行后，会在activeMQ中看到发送的queue 

![1517713617358](/img/posts/1517713617358.png)