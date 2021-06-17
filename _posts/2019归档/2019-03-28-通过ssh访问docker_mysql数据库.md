---
layout:     post
title:      通过ssh访问docker_mysql数据库
date:       2019-03-28
author:     yunzhs
header-img: img/Mayuri with Sakura.jpg
catalog: true
tags:
    - mysql
typora-root-url: ..
typora-copy-images-to: ..\img\posts
---

### 前言

在docker中启动mysql服务后,需要从本地和项目对其进行连接

> docker run -p 3306:3306 --name mysql01 -e MYSQL_ROOT_PASSWORD=root -d 69056560eb44

由于docker所在服务器属于内网服务器,所以需要通过代理服务器来进行连接,因此我们使用ssh通道来进行连接

### Navicat进行连接

![1553754693186](/img/posts/1553754693186.png)

这里填写的基本都是代理服务器的信息,通过代理服务器,我们就相当于已经到了实际服务器,因此接下来相当于一个本地配置.

![1553754868184](/img/posts/1553754868184.png)

和本地的配置一样..

然后:connect successful!!

### spingboot环境下进行连接:

原理和上述方式相同,都是建立ssh连接,再对mysql进行连接

```
package com.sshsql_test.demo.config;

import com.jcraft.jsch.JSch;
import com.jcraft.jsch.Session;

import java.util.Properties;

public class SSHConnection {

//    private final static String S_PATH_FILE_PRIVATE_KEY = "/Users/hdwang/.ssh/id_rsa";
//    private final static String S_PATH_FILE_KNOWN_HOSTS = "/Users/hasee/.ssh/known_hosts";
    private final static String S_PASS_PHRASE = "";
    private final static int LOCAl_PORT = 3307;//本地
    private final static int REMOTE_PORT = 3306;
    private final static int SSH_REMOTE_PORT = 18100;
    private final static String SSH_USER = "root";
    private final static String SSH_PASSWORD = "##########";
    private final static String SSH_REMOTE_SERVER = "111.205.202.87";
    private final static String MYSQL_REMOTE_SERVER = "localhost";

    private Session sesion; //represents each ssh session

    public void closeSSH ()
    {
        sesion.disconnect();
    }

    public SSHConnection () throws Throwable
    {

        JSch jsch = null;

        jsch = new JSch();
//        jsch.setKnownHosts(S_PATH_FILE_KNOWN_HOSTS);
        //jsch.addIdentity(S_PATH_FILE_PRIVATE_KEY);

        sesion = jsch.getSession(SSH_USER, SSH_REMOTE_SERVER, SSH_REMOTE_PORT);

        sesion.setPassword(SSH_PASSWORD);

        Properties config = new Properties();
        config.put("StrictHostKeyChecking", "no");
        sesion.setConfig(config);

        sesion.connect(); //ssh connection established!

        //by security policy, you must connect through a fowarded port
        sesion.setPortForwardingL(LOCAl_PORT, MYSQL_REMOTE_SERVER, REMOTE_PORT);

    }
}
```

**原理是本地开通一个端口去连接服务器上的指定端口,然后本地起来的服务再去连接本地的那个端口**

```
package com.sshsql_test.demo.config;

import org.springframework.context.annotation.Configuration;

import javax.servlet.ServletContextEvent;
import javax.servlet.ServletContextListener;
import javax.servlet.annotation.WebListener;

@WebListener
@Configuration
public class MyContextListener implements ServletContextListener {

    private SSHConnection conexionssh;


    public MyContextListener() {
        super();
    }

    /**
     * @see ServletContextListener#contextInitialized(ServletContextEvent)
     */
    public void contextInitialized(ServletContextEvent arg0) {
        System.out.println("Context initialized ... !");
        try {
            conexionssh = new SSHConnection();
        } catch (Throwable e) {
            e.printStackTrace(); // error connecting SSH server
        }
    }

    /**
     * @see ServletContextListener#contextDestroyed(ServletContextEvent)
     */
    public void contextDestroyed(ServletContextEvent arg0) {
        System.out.println("Context destroyed ... !");
        conexionssh.closeSSH(); // disconnect
    }

}
```

yml配置:

```
spring:
    datasource:
      url: jdbc:mysql://localhost:3307/wb_platform?serverTimezone=Asia/Shanghai&useUnicode=true&useSSL=false&characterEncoding=utf8
      username: root
      password: root
```