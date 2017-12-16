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
typora-copy-images-to: ..\img
---

[TOC]



# IntelliJ IDEA 17和eclipse使用Maven的相关知识

##  一.在idea上创建maven项目

### 1.打开IDEA，创建新项目：

![Snipaste_2017-12-16_11-04-35](/img/Snipaste_2017-12-16_11-04-35.png)

### 2.然后选择Maven，以及选择自己电脑的jdk,并且不要选择create from archetype

![Snipaste_2017-12-16_11-05-20](/img/Snipaste_2017-12-16_11-05-20-3395975050.png)

### 3.接下来自定义GroupId以及ArtifactId，这里只是demo，所以随便命名

![Snipaste_2017-12-16_11-05-52](/img/Snipaste_2017-12-16_11-05-52.png)

### 4.然后自定义项目名,创建完成

![Snipaste_2017-12-16_11-06-02](/img/Snipaste_2017-12-16_11-06-02.png)

### 5.创建完成后的目录结构和样式

![Snipaste_2017-12-16_11-07-13](/img/Snipaste_2017-12-16_11-07-13.png)

### 6.最后在setting中找到maven,设置本地maven工具的目录,并查看相应的本地仓库的位置

![Snipaste_2017-12-16_11-12-10](/img/Snipaste_2017-12-16_11-12-10.png)

## 二.eclipse上的会使用的常用maven插件

### 1.maven-compiler-plugin

​	因为在eclipse上创建maven项目,会自动给项目分配一个jdk1.5,所以需要这个插件来给maven项目制定一个jdk版本.

​	而idea在创建maven项目前,会让你选择要用哪一个的java版本,故不需要导入这个插件.

### 2.tomcat7-maven-plugin

​	是在eclipse上可以自己配置接口以及不用輸項目名就可以直接在網頁中打開,需要運行配置:run as:maven build里進行配置,在goal中輸入,clean tomcat7:run即可直接啟動服務器,但是注意,不能直接點綠三角.否則會直接啟動eclipse自己配置的服務器. 

​	這個功能是idea自帶環境可以直接實現的.

![Snipaste_2017-12-16_17-27-57](/img/Snipaste_2017-12-16_17-27-57.png)

### 3.maven-source-plugin

可以在compile或者install后不光生成相應的可發佈文件,同時生成一份源碼包,供人查看.



## 三 .maven設置中的標籤的一些作用

### 1.依賴範圍<scope></scope>
![Snipaste_2017-12-16_17-38-22](/img/Snipaste_2017-12-16_17-38-22.png)

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

   ![Snipaste_2017-12-16_21-16-06](C:\Users\hasee\Desktop\Snipaste_2017-12-16_21-16-06.png)

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



