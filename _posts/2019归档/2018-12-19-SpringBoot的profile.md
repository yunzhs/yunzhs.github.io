---
layout:     post
title:      SpringBoot的profile
date:       xx18-12-19
author:     yunzhs
header-img: img/Mayuri with Sakura.jpg
catalog: true
tags:
    - SpringBoot
typora-root-url: ..
typora-copy-images-to: ..\img\posts
---

### 前言

spring的profile可以用来进行配置多环境的应用配置,以及相应的常量导入,下面用例子介绍相应的配置.

### 正文

多环境启动配置:

#### application.yml文件中的内容

```yaml
spring:
  profiles: local
  datasource:
    url: jdbc:mysql://xx.xx.xx.xx:3306/database?autoReconnect=true&useUnicode=true&characterEncoding=utf8&zeroDateTimeBehavior=convertToNull&useSSL=false
    username: database
    password: database
    db-name: database #用来搜集数据库的所有表
    filters: wall,mergeStat
    
spring:
  profiles: test
  datasource:
    url: jdbc:mysql://xx.xxx.xx.47:3306/database?autoReconnect=true&useUnicode=true&characterEncoding=utf8&zeroDateTimeBehavior=convertToNull&useSSL=false
    username: database
    password: database
    db-name: database #用来搜集数据库的所有表
    filters: wall,mergeStat
  redis:
    database: 8
    jedis:
      pool:
        max-idle: xx0
        max-wait: -1s
        max-active: 50
        min-idle: 0
    timeout: 5000s
    password: Dkjkjkl17K8NOnP
    sentinel:
      master: mymaster
      nodes: xx.xxx.xx.11:9098,xx.xxx.xx.12:9098    
constant:
  dlj-url: http://xx.xxx.xx.52:8xx1
eureka:
  instance:
    prefer-ip-address: true
    instance-id: ${spring.cloud.client.ip-address}:${server.port}
  client:
    service-url:
#          defaultZone: http://xx.xxx.xx.136:5438/eureka,http://xx.xxx.xx.137:5438/eureka,http://xx.xxx.xx.138:5438/eureka
          defaultZone: http://c371501xx61-2:5438/eureka,http://c371501xx61-3:5438/eureka,http://c371501xx61-4:5438/eureka


      
spring:
  profiles: produce
  datasource:
    url: jdbc:mysql://xx.xxx.xx.130:3306/database?autoReconnect=true&useUnicode=true&characterEncoding=utf8&zeroDateTimeBehavior=convertToNull&useSSL=false
    username: database
    password: database
    db-name: database #用来搜集数据库的所有表
    filters: wall,mergeStat
eureka:
  instance:
    prefer-ip-address: true
    instance-id: ${spring.cloud.client.ip-address}:${server.port}
  client:
    service-url:
          defaultZone: http://xx.xxx.xx.136:5438/eureka,http://xx.xxx.xx.137:5438/eureka,http://xx.xxx.xx.138:5438/eureka
```

不同环境下可以分别命名,local,test,produce.

可以在相应的配置下进行数据库,redis,常量,eureka等的配置.

注入profile中的常量内容:

```
    private static String dljUrl;
    @Value("${constant.dlj-url}")
    public void setDljUrl(String dljUrl) {
        HttpUtil.dljUrl = dljUrl;
    }
```

#### pom文件中的内容

```
<profiles>
    <profile>
        <id>local</id>
        <properties>
            <spring.active>local</spring.active>
        </properties>

    </profile>
    <profile>
        <id>test</id>
        <properties>
            <spring.active>test</spring.active>
        </properties>
    </profile>
    <profile>
        <id>produce</id>
        <properties>
            <spring.active>produce</spring.active>
        </properties>
        <activation>
            <activeByDefault>true</activeByDefault>
        </activation>
    </profile>
</profiles>
```

正常启动时可以根据yml文件中的配置来决定选址哪一个配置环境,也可以根据pom文件中的`<activeByDefault>`  来指定以哪一个环境来启动.

```
spring:
  profiles:
    active: test
```

