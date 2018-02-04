---
layout:     post
title:      基于SpringBoot的一个短信验证小Demo
date:       2018-1-28
author:     yunzhs
header-img: img/Archer.jpg
catalog: true
tags:
    - SpringBoot
typora-root-url: ..
typora-copy-images-to: ..\img\posts
---

​                                                                                                                                                                                                                                                                                                                        为什么要用redis而不用session,因为session一个服务器每次会话只能有一个,在生产环境时,我们使用的是ngaix服务器,可能会有多台机器对服务器发起请求,这样如果使用session便不能满足服务器的需求,

同时,为了满足短信验证码的时效功能,我们需要对数据设置时效性,而redis对于时效性的设置易于并且优于session,断网重连后session也会被重置,对用户很不友好

redis 有个setex设置键值 ,可以设置过期时间 PERSIST 则设置为持久的 

ttl可以返回-1表示未设置,返回-2表示已过期

​                                                                                                                                                                                                                                                                    