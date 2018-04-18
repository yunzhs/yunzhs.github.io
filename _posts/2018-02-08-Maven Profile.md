---
layout:     post
title:      Maven Profile
subtitle:   
date:       2017-12-5
author:     yunzhs
header-img: img/Dyanna the Luna.jpg
catalog: true
tags:
    - Maven Profile
typora-root-url: ..
typora-copy-images-to: ..\img\posts
---

## 一.什么是MavenProfile

​	在我们平常的java开发中，会经常使用到很多配制文件（xxx.properties，xxx.xml），而当我们在本地开发（dev），测试环境测试（test），线上生产使用（product）时，需要不停的去修改这些配制文件，次数一多，相当麻烦。现在，利用maven的filter和profile功能，我们可实现在编译阶段简单的指定一个参数就能切换配制，提高效率，还不容易出错.

​	profile可以让我们定义一系列的配置信息，然后指定其激活条件。这样我们就可以定义多个profile，然后每个profile对应不同的激活条件和配置信息，从而达到不同环境使用不同配置信息的效果。



## 二. Maven Profile入门

```
 <properties>
  	<port>9105</port>
  </properties>
  <build>  
	  <plugins>	     
	      <plugin>
				<groupId>org.apache.tomcat.maven</groupId>
				<artifactId>tomcat7-maven-plugin</artifactId>
				<version>2.2</version>
				<configuration>
					<!-- 指定端口 -->
					<port>${port}</port>
					<!-- 请求路径 -->
					<path>/</path>
				</configuration>
	  	  </plugin>
	  </plugins>  
    </build>

```

这是一个pom.xml文件

如果这个端口在开发时使用9105，如果在生产环境（或其他环境）为9205呢？如何解决值的动态切换呢？

这时我们修改pom.xml，增加profile定义

```
  <profiles>
  	<profile>
  		<id>dev</id>
  		<properties>
  			<port>9105</port>
  		</properties>
  	</profile>
  	<profile>
  		<id>pro</id>
  		<properties>
  			<port>9205</port>
  		</properties>
  	</profile>  
  </profiles>

```

执行命令 tomcat7:run -Ppro  发现以9205端口启动

执行命令 tomcat7:run -Pdev  发现以9105端口启动

-P 后边跟的是profile的id

如果我们只执行命令tomcat7:run ,也是以9105启动，因为我们一开始定义的变量值就是9105，就是在不指定profileID时的默认值.

## 三.实例

（1）我们在dao工程中src/main/resources下创建filter文件夹

（2）filter文件夹下创建db_dev.properties ，用于配置开发环境用到的数据库

（3）filter文件夹下创建db_pro.properties  

（4）修改properties下的db.properties

#### 定义Profile

修改pom.xml 

```
  <properties>
  		<env>dev</env>
  </properties>
  <profiles>
  	<profile>
  		<id>dev</id>
  		<properties>
  			<env>dev</env>
  		</properties>
  	</profile>    
  	<profile>
  		<id>pro</id>
  		<properties>
  			<env>pro</env>
  		</properties>
  	</profile>
  </profiles>

```



这里我们定义了2个profile，分别是开发环境和生产环境

#### 资源过滤与变量替换

修改pom.xml ，在build节点中添加如下配置

```
  	<filters>
  		<filter>src/main/resources/filters/db_${env}.properties</filter>
  	</filters>
  	<resources>
  		<resource>
  			<directory>src/main/resources</directory>
  			<filtering>true</filtering>
  		</resource>  		
  	</resources>

```

这里我们利用filter实现对资源文件(resouces) 过滤 
maven filter可利用指定的xxx.properties中对应的key=value对资源文件中的 \$ {key}进行替换，最终把你的资源文件中的username=${key}替换成username=value 

#### 打包

在pinyougou-dao 工程 执行命令：package -P pro ,  解压生成的jar包，观察db.properties配置文件内容，已经替换为生产环境的值。

在pinyougou-sellergoods-service工程 执行命令 pageage  ，解压生成的war包里的pinyougou-dao的jar包，发现也是生成环境的值。

#### 测试运行

【1】连接生产数据库

（1）在dao 工程执行命令：install-P pro

（2）在service：执行命令：tomcat7:run 

（3）在web ：  执行命令：tomcat7:run

【2】连接开发数据库

（1）在dao 工程执行命令：install-P dev  (或 install 默认是dev  )

（2）在service：执行命令：tomcat7:run 

（3）在web ：  执行命令：tomcat7:run