---
layout:     post
title:      IntelliJ IDEA 17和eclipse使用Maven的相关知识
date:       2017-12-16
author:     yunzhs
header-img: img/tag-bg.jpg
catalog: true
tags:
    - maven
typora-root-url: ..
typora-copy-images-to: ..\img\posts
---

[TOC]



# IntelliJ IDEA 17和eclipse使用Maven的相关知识

##  一.在idea上创建maven项目

### 1.打开IDEA，创建新项目：

![Snipaste_2017-12-16_11-04-35](/img/posts/Snipaste_2017-12-16_11-04-35.png)

### 2.然后选择Maven，以及选择自己电脑的jdk,并且不要选择create from archetype

![Snipaste_2017-12-16_11-05-20](/img/posts/Snipaste_2017-12-16_11-05-20-3395975050.png)

### 3.接下来自定义GroupId以及ArtifactId，这里只是demo，所以随便命名

![Snipaste_2017-12-16_11-05-52](/img/posts/Snipaste_2017-12-16_11-05-52.png)

### 4.然后自定义项目名,创建完成

![Snipaste_2017-12-16_11-06-02](/img/posts/Snipaste_2017-12-16_11-06-02.png)

### 5.创建完成后的目录结构和样式

![Snipaste_2017-12-16_11-07-13](/img/posts/Snipaste_2017-12-16_11-07-13.png)

### 6.最后在setting中找到maven,设置本地maven工具的目录,并查看相应的本地仓库的位置

![Snipaste_2017-12-16_11-12-10](/img/posts/Snipaste_2017-12-16_11-12-10.png)

### 7.将MAVEN的默认jdk版本设置为1.8

​	在pom.xml文件中配置:

```
<properties>
    <maven.compiler.source>1.8</maven.compiler.source>
    <maven.compiler.target>1.8</maven.compiler.target>
</properties>
```

### 8.添加WEB配置

选择框架支持,并添加web.application

![Snipaste_2017-12-22_21-36-42](/img/posts/Snipaste_2017-12-22_21-36-42.png)

### 9.处理java版本过期问题

- Check java version in your **pom.xml**([here](https://stackoverflow.com/questions/16723533/modify-pom-xml-to-include-jdk-compiler-version) you can find how to do it). Also check java version in **Project Structure**. And the last what you can do - check compiler version e.g.

![enter image description here](https://i.stack.imgur.com/itRLK.jpg)



- I did all of the above and still had one instance of the warning:

  ```
  Warning:java: source value 1.5 is obsolete and will be removed in a future release
  ```

  I went into my **project_name.iml** file and replaced the following tag:

  ```
  <component name="NewModuleRootManager" LANGUAGE_LEVEL="JDK_1_5" inherit-compiler-output="false">
  ```

  with:

  ```
  <component name="NewModuleRootManager" LANGUAGE_LEVEL="JDK_1_8" inherit-compiler-output="false">
  ```

  And voila, no more error message. Hope this helps someone.

- 最后感觉还是直接在里面导入插件比较实用

  ​



## 二.eclipse上的会使用的常用maven插件

### 1.maven-compiler-plugin插件

​	因为在eclipse上创建maven项目,会自动给项目分配一个jdk1.5,所以需要这个插件来给maven项目制定一个jdk版本.

​	~~而idea在创建maven项目前,会让你选择要用哪一个的java版本,故不需要导入这个插件.~~

```
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-compiler-plugin</artifactId>
    <version>3.1</version>
    <configuration>
        <source>1.8</source>
        <target>1.8</target>
        <encoding>utf-8</encoding>
    </configuration>
</plugin>
```

### 2.tomcat7-maven-plugin

​	是在eclipse上可以自己配置接口以及不用輸項目名就可以直接在網頁中打開,需要運行配置:run as:maven build里進行配置,在goal中輸入,clean tomcat7:run即可直接啟動服務器,但是注意,不能直接點綠三角.否則會直接啟動eclipse自己配置的服務器. 

​	這個功能是idea自帶環境可以直接實現的.

![Snipaste_2017-12-16_17-27-57](/img/posts/Snipaste_2017-12-16_17-27-57.png)

### 3.maven-source-plugin

可以在compile或者install后不光生成相應的可發佈文件,同時生成一份源碼包,供人查看.



## 三 .maven設置中的標籤的一些作用

### 1.依賴範圍<scope></scope>
![Snipaste_2017-12-16_17-38-22](/img/posts/Snipaste_2017-12-16_17-38-22.png)

servlet為什麼是provided?
​	因為在實際的環境中,打包過程中tomcat已經自帶這玩意兒了,再被打包的話會發生衝突.

### 2.依賴控制<option></option>

​	<optional>true</optional> 表示可以想下傳遞,若是false的話則表示不能向下傳遞

### 3.依賴排除<exclusions></exclusions>

​	是與dependence相反的標籤.在dependency中定義,可以將依賴中的不需要的jar排除出去



## 四.maven的三大生命週期

-  clean：清理项目的

-  default：构建项目的

-  site：生成项目站点的

   ![Snipaste_2017-12-16_21-16-06](/img/posts/Snipaste_2017-12-16_21-16-06.png)

   maven項目的運行需要在run configurations進行,在goal框中輸入相關的指令,不同指令間可以使用空格隔開

   常見的指令主要有:

   clean:移除所有上一次构建生成的文件 
   compile:對項目中的文件進行編譯,在target中生成相應的class文件
   run:運行項目
   package: 接受编译好的代码，打包成可发布的格式，如 JAR ,WAR
   install:将包安装至本地仓库，以让其它项目依赖




## 五.eclipse上pom第一行報錯的解決方法
- 首先确定你的电脑是否可以连接网络。
- 如果可以连接网络，在maven的本地库的路径下执行以下命令： 

​       `for /r %i in (*.lastUpdated) do del %i`

- 最后，尝试刷新maven工程，看是否可以成功。


## 六.解决导入pom文件后SLF4J和log4j报错的问题

SLF4J: Failed to load class "org.slf4j.impl.StaticLoggerBinder".
SLF4J: Defaulting to no-operation (NOP) logger implementation
SLF4J: See http://www.slf4j.org/codes.html#StaticLoggerBinder for further details.

造成原因是SLF4J log4j版本不匹配

首先看看你工程中的sl4j-api的版本（比如我的是1.5.11），然后在<http://mvnrepository.com/>搜索slf4j-log4j12，会出现SLF4J LOG4J 12 Binding，点击进入，会有很多版本的slf4j-log4j12，我们点击1.5.11版本的slf4j-log4j12进入详细信息页面，查看依赖的log4j，这个版本的slf4j-log4j12依赖的是1.2.14版本的log4j。

所以，**我们在我们的工程中添加1.5.11版本的slf4j-log4j12和1.2.14版本的log4j，问题完美解决。**

**另外把slf4j-log4j12中的test去掉才行**