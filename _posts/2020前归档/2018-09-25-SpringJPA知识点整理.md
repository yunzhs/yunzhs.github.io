---
layout:     post
title:      SpringJPA知识点整理
date:       2018-09-25
author:     yunzhs
header-img: img/post-bg-rwd.jpg
catalog: true
tags:
    - 随笔
typora-root-url: ..
typora-copy-images-to: ..\img\posts
---

### JPA的好处

1.只需建一个repository,就可以完成各种操作,无论是用函数,还是自己写sql语句,都可以快速完成

2.可以非常舒服的实现一对多,多对多的sql查询问题,只需在实体类中,加上相应的onetomany标签,并确定相应的外键,就可以自动查询相关表,并放在相应的实体类下.

3.添加字段不需要到数据库中去添加,只需要在实体类中添加即可,运行一遍程序,就能在表中自动生成相应的字段.



### JPA的表重置

spring JPA默认的数据库表格式,如果原有表不符合规定格式,系统会自动生成一个符合格式的表结构,不会有数据.

不符合的表

![1537839306298](/img/posts/1537839306298.png)

自动生成并导入数据后的表

![1537839345658](/img/posts/1537839345658.png)

### @OneToMany一对多遇到的问题

```
@OneToMany(fetch = FetchType.EAGER)
@JoinColumn(name = "ORDERNO", referencedColumnName = "ORDERNO")
```

在一个表的实体类中,如果只有一个一对多,那么关联表用list是没有问题,但是如果有两个表,那么就会报这样的问题:

```
org.hibernate.loader.MultipleBagFetchException: cannot simultaneously fetch multiple bags: [com.zjx.model.dataModel.Ordertable.ticketcartables, com.zjx.model.dataModel.Ordertable.ticketpersontabless]
```

需改为:

```
private Set<Ticketcartable> ticketcartables=Sets.newLinkedHashSet();
private Set<Ticketpersontable> ticketpersontabless=Sets.newLinkedHashSet();
```

原因:

```
A list, if there is no index column specified, will just be handled as a bag by Hibernate (no specific ordering).

One notable difference in the handling of Hibernate is that you can't fetch two different lists in a single query. For example, if you have a Person entity having a list of contacts and a list of addresses, you won't be able to use a single query to load persons with all their contacts and all their addresses. The solution in this case is to make two queries (which avoids the cartesian product), or to use a Set instead of a List for at least one of the collections.
```

无法在一个查询中获取两个不同的列表,之所以这样是为了避免由于笛卡尔积而产生相同数值不同排列,从而产生多个结果

