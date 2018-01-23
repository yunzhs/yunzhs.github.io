---
layout:     post
title:      在虚拟机上配置linux镜像的网络
date:       2018-1-3
author:     yunzhs
header-img: img/Fate of Princess.jpg
catalog: true
tags:
    - linux
typora-root-url: ..
typora-copy-images-to: ..\img\posts
---

# linux下yum安装redis以及使用

### **1、yum install redis      --查看是否有redis   yum 源**

**2、yum install epel-release    --下载fedora的epel仓库**

**3、 yum install redis    -- 安装redis数据库**

**4、service redis start  Redirecting to /bin/systemctl start redis.service   --开启redis服务**

​      **redis-server /etc/redis.conf   --开启方式二**

**5、ps -ef | grep redis   -- 查看redis是否开启**

**6、redis-cli       -- 进入redis服务**

**7、redis-cli  shutdown      --关闭服务**

**8、开放端口6379、6380的防火墙**

​     **/sbin/iptables -I INPUT -p tcp --dport 6379  -j ACCEPT   开启6379   **

​     **/sbin/iptables -I INPUT -p tcp --dport 6380 -j ACCEPT  开启6380**

​     **/etc/rc.d/init.d/iptables save                           保存**

  **连接数据库./redis-cli -h 192.168.217.131 -p 6379**

**9、使用redis  desktop manager连接redis** 