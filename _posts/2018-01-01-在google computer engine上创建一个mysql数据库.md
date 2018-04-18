---
layout:     post
title:      在google computer engine上创建一个mysql数据库
subtitle:   
date:       2017-07-1
author:     yunzhs
header-img: img/Dyanna the Luna.jpg
catalog: true
tags:
    - linux
    - mysql
typora-root-url: ..
typora-copy-images-to: ..\img\posts
---

# Centos7安装Mysql

### 一、安装

在Centos7以前，正常安装是使用命令

```
yum install -y mysql-server mysql mysql-devel
```

但是，Centos7默认安装的数据库是MariaDB，虽然MariaDB是Mysql的一个分支，并且完全兼容Mysql的API及命令行。

通过以下步骤还是可使用yum来进行安装

####  0、新装系统安装需要获得root权限先执行此步

```
yum -y install wget 
```

#### 1、下载Mysql的repo源

```
$ wget http://repo.mysql.com/mysql-community-release-el7-5.noarch.rpm

```

#### 2、安装mysql-community-release-el7-5.noarch.rpm包

```
$ rpm -ivh mysql-community-release-el7-5.noarch.rpm

```

#### 3、安装mysql

```
$ yum install -y mysql-server mysql mysql-devel

```

### 二、重置密码

因为Mysql的默认密码是空，所以有必要设置一个密码。

#### 1、登录Mysql

```
$ mysql -u root

```

风险提示：这里可能会报如下错误

> Can't connect to local MySQL server through socket '/var/lib/mysql/mysql.sock'(2)

不用慌张，只是权限问题而已，只需要为目录 /var/lib/mysql 赋权，然后重启Mysql服务即可

```
$ chmod 777 /var/lib/mysql
$ service mysqld restart

```

#### 2、设置新密码

```
mysql> use mysql;
mysql> update user set password=password('123') where user = 'root';
mysql> flush privileges;
mysql> exit;

```

#### 3、重启服务

```
$ service mysqld restart

```

### 三、设置字符编码

安装数据库就是为了对接程序，为了保证系统不出现乱码的现象，统一字符编码是个非常重要的环节，目前行内的习惯是统一使用 utf8 编码

#### 1、查看当前编码

登录，前面已经设置了密码，需要使用密码进行登录

```
$ mysql -u root -p

```

查看编码

```
mysql> show variables like "%char%";

```

可以看到其中有几个配置默认并不是使用 utf8 编码

#### 2、修改配置文件

Mysql的配置文件默认位置为： /etc/my.cnf，编辑该文件，添加如下内容

```
[client]
default-character-set=utf8

[mysqld]
character-set-server=utf8

```

#### 3、重启Mysql服务

```
$ service mysqld restart

```

可以再次执行步骤1，查看编码是否已经全部设置为 utf8

### 四、设置开机启动

编辑文件 /etc/rc.local，添加以下内容

```
service mysqld start

```

### 五、设置远程连接

Mysql默认使用本机进行连接，如果需要远程连接，也需要进行一番设置

#### 1、登录Mysql

```
$ mysql -u root -p
```

#### 2、授权

```
mysql>GRANT ALL PRIVILEGES ON *.* TO root@'%' IDENTIFIED BY '123' WITH GRANT OPTION;
```

@后面为适用的主机,若是%则表示全部适用,123表示登陆密码

#### 3、刷新授权

```
mysql> flush privileges;
```