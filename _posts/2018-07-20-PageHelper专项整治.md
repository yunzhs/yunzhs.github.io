---
layout:     post
title:      PageHelper专项整治
date:       2018-07-20
author:     yunzhs
header-img: img/post-bg-rwd.jpg
catalog: true
tags:
    - 随笔
typora-root-url: ..
typora-copy-images-to: ..\img\posts
---

##  一.前言

PageHelper是一个免费开源的分页插件,这里主要学习用它来为mybatis作分页.



## 二.PageHelper的maven依赖及插件配置

maven的pom依赖

```
<dependency>
	<groupId>com.github.pagehelper</groupId>
	<artifactId>pagehelper</artifactId>
	<version>4.1.6</version>
</dependency>
```

配置参数,可以不设,有默认值

```
<!-- com.github.pagehelper为PageHelper类所在包名 -->
<plugin interceptor="com.github.pagehelper.PageHelper">
	<property name="dialect" value="mysql" />
	<!-- 该参数默认为false -->
	<!-- 设置为true时，会将RowBounds第一个参数offset当成pageNum页码使用 -->
	<!-- 和startPage中的pageNum效果一样 -->
	<property name="offsetAsPageNum" value="false" />
	<!-- 该参数默认为false -->
	<!-- 设置为true时，使用RowBounds分页会进行count查询 -->
	<property name="rowBoundsWithCount" value="true" />
 
	<!-- 设置为true时，如果pageSize=0或者RowBounds.limit = 0就会查询出全部的结果 -->
	<!-- （相当于没有执行分页查询，但是返回结果仍然是Page类型） <property name="pageSizeZero" value="true"/> -->
 
	<!-- 3.3.0版本可用 - 分页参数合理化，默认false禁用 -->
	<!-- 启用合理化时，如果pageNum<1会查询第一页，如果pageNum>pages会查询最后一页 -->
	<!-- 禁用合理化时，如果pageNum<1或pageNum>pages会返回空数据 -->
	<property name="reasonable" value="true" />
	<!-- 3.5.0版本可用 - 为了支持startPage(Object params)方法 -->
	<!-- 增加了一个`params`参数来配置参数映射，用于从Map或ServletRequest中取值 -->
	<!-- 可以配置pageNum,pageSize,count,pageSizeZero,reasonable,不配置映射的用默认值 -->
	<!-- 不理解该含义的前提下，不要随便复制该配置 <property name="params" value="pageNum=start;pageSize=limit;"/> -->
</plugin>

```

dialect：标识是哪一种数据库，设计上必须。

offsetAsPageNum：将RowBounds第一个参数offset当成pageNum页码使用，这就是上面说的一参两用，个人觉得完全没必要，offset = pageSize * pageNum就搞定了，何必混用参数呢？

rowBoundsWithCount：设置为true时，使用RowBounds分页会进行count查询，个人觉得完全没必要，实际开发中，每一个列表分页查询，都配备一个count数量查询即可。

reasonable：value=true时，pageNum小于1会查询第一页，如果pageNum大于pageSize会查询最后一页 ，个人认为，参数校验在进入Mybatis业务体系之前，就应该完成了，不可能到达Mybatis业务体系内参数还带有非法的值。

这么一来，我们只需要记住 dialect = mysql 一个参数即可，其实，还有下面几个相关参数可以配置。

autoDialect：true or false，是否自动检测dialect。

autoRuntimeDialect：true or false，多数据源时，是否自动检测dialect。

closeConn：true or false，检测完dialect后，是否关闭Connection连接。

上面这3个智能参数，不到万不得已，我们不应该在系统中使用，我们只需要一个dialect = mysql 或者 dialect = oracle就够了，如果系统中需要使用，还是得问问自己，是否真的非用不可。

## 三.PageHelper的两种使用方式

### 1.直接通过RowBounds参数完成分页查询

rowbounds方法,第一个参数代表起始行,第二个参数表示当前页实现多少行数据

```
List<Student> list = studentMapper.find(new RowBounds(0, 10));
Page page = ((Page) list;
```

### 2.PageHelper.startPage()静态方法 

```
//获取第1页，10条内容，默认查询总数count
    PageHelper.startPage(1, 10);
//紧跟着的第一个select方法会被分页
    List<Country> list = studentMapper.find();
    Page page = ((Page) list;
```

