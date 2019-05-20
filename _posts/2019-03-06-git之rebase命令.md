---
layout:     post
title:      git一些原理以及rebase命令
date:       2019-03-06
author:     yunzhs
header-img: img/Mayuri with Sakura.jpg
catalog: true
tags:
    - git
typora-root-url: ..
typora-copy-images-to: ..\img\posts
---

### 前言

最近项目新使用了gitLab做版本管理,于是我又回归了git,一日,我在修改完代码进行commit and push的操作,push操作后提示我与服务器版本冲突(emmmm,我忘了先拉了),紧接着idea提示我可以进行merge或者rebase操作(看它的提示貌似推荐我使用rebase),我以前也经常见这个rebase,于是就点了打算试试(反正有git怕什么),结果悲剧发生了,我除了提交的代码,其他代码都消失 了~~~,是真的消失了哦,于是我就开始了rebase的学习.

### 详解

![1551860843112](/img/posts/1551860843112.png)

首先看这张图左边的曲线图,可以看见整体的一个提交情况,当基于服务器版本进行开发时,会一直沿着主干前进,但如果基于服务器版本,然而服务器版本又有了新的变化时就会产生分支,

在本地进行提交后,如果是落后于版本,在push时,会执行merge操作,拉取服务器版本来结束分支,如上图的分支汇流

rebase就是在这样的背景下产生的.

![1551861653872](/img/posts/1551861653872.png)

可以看到c5 c6加到了c4后面,整体变成一条直线,因此,执行rebase命令后,mywork会撤销自己的commit,从而在实现这一波操作.