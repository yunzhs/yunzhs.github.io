---
layout:     post
title:      在IDEA配置mybatis接口代理idea Invalid bound statement (not found)
subtitle:   
date:       2017-12-19
author:     yunzhs
header-img: img/Dyanna the Luna.jpg
catalog: true
tags:
    - idea
    - mybatis
typora-root-url: ..
typora-copy-images-to: ..\img\posts
---

# 在IDEA配置mybatis接口代理,出现idea Invalid bound statement (not found)错误

##1.前言

学习mybatis的过程中，测试mapper自动代理的时候一直出错，在eclipse中可以正常运行，而同样的代码在idea中却无法成功。虽然可以继续调试，但心里总是纠结原因。百度了好久，终于找到一个合适的原因。参考：<http://blog.csdn.net/z69183787/article/details/48933481>；

IDEA的maven项目中，默认源代码目录下的xml等资源文件并不会在编译的时候一块打包进classes文件夹，而是直接舍弃掉。

如果使用的是Eclipse，Eclipse的src目录下的xml等资源文件在编译的时候会自动打包进输出到classes文件夹。Hibernate和Spring有时会将配置文件放置在src目录下，编译后要一块打包进classes文件夹，所以存在着需要将xml等资源文件放置在源代码目录下的需求。

## 2.在maven的pom文件中配置过滤,使其自动加载xml文件

```
<build>
    <plugins>
      <plugin>
        <groupId>org.mortbay.jetty</groupId>
        <artifactId>maven-jetty-plugin</artifactId>
        <version>6.1.7</version>
        <configuration>
          <connectors>
            <connector implementation="org.mortbay.jetty.nio.SelectChannelConnector">
              <port>8888</port>
              <maxIdleTime>30000</maxIdleTime>
            </connector>
          </connectors>
          <webAppSourceDirectory>${project.build.directory}/${pom.artifactId}-${pom.version}</webAppSourceDirectory>
          <contextPath>/</contextPath>
        </configuration>
      </plugin>
    </plugins>

    <!--这里进行配置后会自动的加载mapper.xml文件　:配置Maven 对resource文件 过滤 -->
    <resources>
      <resource>
        <directory>src/main/resources</directory>
        <includes>
          <include>**/*.properties</include>
          <include>**/*.xml</include>
        </includes>
        <filtering>true</filtering>
      </resource>
      <resource>
        <directory>src/main/java</directory>
        <includes>
          <include>**/*.properties</include>
          <include>**/*.xml</include>
        </includes>
        <filtering>true</filtering>
      </resource>
    </resources>

  </build>
```

## 3.可以在resource文件中直接建一个和在java下目录结构相同的包,把xml放进去,效果与放在一起一样.	

![Snipaste_2017-12-19_09-31-27](/img/posts/Snipaste_2017-12-19_09-31-27.png)