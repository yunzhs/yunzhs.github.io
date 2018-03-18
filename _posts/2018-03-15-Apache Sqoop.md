---
layout:     post
title:      Apache Sqoop安装使用
date:       2018-3-15
author:     yunzhs
header-img: img/Fate of Princess.jpg
catalog: true
tags:
    - bigdata相关
typora-root-url: ..
typora-copy-images-to: ..\img\posts
---

## 一.简介

​	Sqoop 是 是 p Hadoop 和关系数据库服务器之间传送数据的一种工具。它是用来从关系数据库如：MySQL，

Oracle 到 Hadoop 的 HDFS，并从 Hadoop 的文件系统导出数据到关系数据库。由 Apache 软件基金会提供。

​	Sqoop：“SQL 到 Hadoop 和 Hadoop 到 SQL”。

![52110463129](/img/posts/1521104631295.png)

Sqoop 工作机制是将导入或导出命令翻译成 mapreduce 程序来实现。

在翻译出的 mapreduce 中主要是对 inputformat 和 outputformat 进行定制。

## 二.sqoop安装

安装 sqoop 的前提是已经具备 java 和 hadoop 的环境。

最新稳定版： 1.4.6

解压

```
tar -zxvf sqoop-1.4.6.bin__hadoop-2.0.4-alpha.tar.gz
```

配置文件修改：

```
cd $SQOOP_HOME/conf

mv sqoop-env-template.sh sqoop-env.sh

vi sqoop-env.sh

export HADOOP_COMMON_HOME=/export/servers/hadoop/

export HADOOP_MAPRED_HOME=/export/servers/hadoop/

export HIVE_HOME=/export/servers/hive

```

加入 mysql 的 jdbc 驱动包

```
cp /export/servers/hive/lib/mysql-connector-java-5.1.34.jar ../lib/
```

验证启动

```
bin/sqoop list-databases --connect jdbc:mysql://localhost:3306/ --username root --password 123
```

本命令会列出所有 mysql 的数据库。
到这里，整个 Sqoop 安装工作完成。

## 三.sqoop导入

“导入工具”导入单个表从 RDBMS 到 HDFS。表中的每一行被视为 HDFS 的记录。所有记录都存储为文本文件的文

本数据（或者 Avro、sequence 文件等二进制数据）。

下面的语法用于将数据导入 HDFS。

$ sqoop import (generic-args) (import-args)

Sqoop 测试表数据

在 mysql 中创建数据库 userdb，然后执行参考资料中的 sql 脚本：

创建三张表: emp emp_add emp_conn。

### 1. 导入 mysql 表数据到 ==HDFS==

下面的命令用于从 MySQL 数据库服务器中的 emp 表导入 HDFS。

```
bin/sqoop import \
--connect jdbc:mysql://node-01:3306/yunzhs \
--username root \
--password 123 \
--target-dir /sqoopresult \
--table emp --m 1
```

其中--target-dir 可以用来指定导出数据存放至 HDFS 的目录；

mysql jdbc url 请使用 ip 地址。

为了验证在 HDFS 导入的数据，请使用以下命令查看导入的数据：

```
hdfs dfs -cat /sqoopresult/part-m-00000
```

可以看出它会用逗号,分隔 emp 表的数据和字段。

--m 1 表示使用几个mapreduce来执行

![52110731475](/img/posts/1521107314758.png)

### 2.导入 mysql 表数据到==HIVE==

将关系型数据的表结构复制到 hive 中

```
bin/sqoop create-hive-table \
--connect jdbc:mysql://node-01:3306/yunzhs \
--table emp_add \
--username root \
--password 123 \
--hive-table momo.emp_add_sp
```

从关系数据库导入文件到 hive 中

```
bin/sqoop import \
--connect jdbc:mysql://node-01:3306/yunzhs \
--username root \
--password 123 \
--table emp_add \
--hive-table momo.emp_add_sp \
--hive-import \
--m 1
```

--table  ,--hive-table  分别指mysql中表名,和hive中的数据库加表名

### 3.导入表数据子集

​	--where 可以指定从关系数据库导入数据时的查询条件。它执行在各自的数据库服务器相应的 SQL 查询，并

将结果存储在 HDFS 的目标目录。

```
bin/sqoop import \
--connect jdbc:mysql://node-21:3306/sqoopdb \
--username root \
--password hadoop \
--where "city ='sec-bad'" \
--target-dir /wherequery \
--table emp_add --m 1
```

复杂查询条件:

```
bin/sqoop import \
--connect jdbc:mysql://node-21:3306/sqoopdb \
--username root \
--password hadoop \
--target-dir /wherequery12 \
--query 'select id,name,deg from emp WHERE id>1203 and $CONDITIONS' \
--split-by id \
--fields-terminated-by '\t' \
--m 1
```

根据查询语句来进行数据导入

### 4.增量导入

增量导入是仅导入新添加的表中的行的技术。

--check-column (col) 用来作为判断的列名，如 id

--incremental (mode) append：追加，比如对大于 last-value 指定的值之后的记录进行追加导入。

--lastmodified：最后的修改时间，追加 last-value指定的日期之后的记录

--last-value (value) 指定自从上次导入后列的最大值（大于该指定的值），也可以自己设定某一值

假设新添加的数据转换成 emp 表如下：

![52110921964](/img/posts/1521109219643.png)

1206, satish p, grp des, 20000, GR

下面的命令用于在 EMP 表执行增量导入：

```
bin/sqoop import \
--connect jdbc:mysql://node-21:3306/sqoopdb \
--username root \
--password hadoop \
--table emp --m 1 \
--incremental append \
--check-column id \
--last-value 1205
```

## 四.Sqoop 导出

将数据从 HDFS 导出到 RDBMS 数据库导出前，**目标表**必须存在于目标数据库中。

默认操作是从将文件中的数据使用INSERT语句插入到表中，更新模式下，是生成UPDATE语句更新表数据。

以下是 export 命令语法：

$ sqoop export (generic-args) (export-args)

### 1.导出 HDFS 数据到 mysql

数据是在 HDFS 中“emp/”目录的 emp_data 文件中：

```
1201,gopal,manager,50000,TP
1202,manisha,preader,50000,TP
1203,kalil,php dev,30000,AC
1204,prasanth,php dev,30000,AC
1205,kranthi,admin,20000,TP
1206,satishp,grpdes,20000,GR
```

首先需要手动创建 mysql 中的目标表：

```
CREATE TABLE employee (
id INT NOT NULL PRIMARY KEY,
name VARCHAR(20),
deg VARCHAR(20),
salary INT,
dept VARCHAR(10));
```

导出命令：

```
bin/sqoop export \
--connect jdbc:mysql://node-21:3306/sqoopdb \
--username root \
--password hadoop \
--table employee \
--export-dir /emp/emp_data
```

还可以用下面命令指定输入文件的分隔符

```
--input-fields-terminated-by '\t'
```



