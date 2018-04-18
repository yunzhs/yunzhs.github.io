---
layout:     post
title:      angularJS的学习使用
date:       2017-08-12
author:     yunzhs
header-img: img/Archer.jpg
catalog: true
tags:
    - angularJS
    - 前端
typora-root-url: ..
typora-copy-images-to: ..\img\posts
---

## 一.angularJS简介

AngularJS诞生于2009年，由Misko Hevery 等人创建，后为Google所收购。是一款优秀的前端JS框架，已经被用于Google的多款产品当中。AngularJS有着诸多特性，最为核心的是：MVVM、模块化、[自动化双向数据绑定](http://www.angularjs.net.cn/tutorial/10.html)、语义化标签、[依赖注入](http://www.angularjs.net.cn/tutorial/17.html)等等。

ng-if

ng-select2

ng-option

$watch() 监视变量

![Snipaste_2018-01-16_18-33-34](/img/posts/Snipaste_2018-01-16_18-33-34.png)

研究了一下,model真正决定要在select中要显示的数据,它绑定的是as前面的那个值,并根据as前的值变换,且负责与后台数据交互,而下拉框显示的内容则是as之后的内容.与数据库中的as用法类似,as是可有可无的!

可以通过设置model来为select选择一个初始值



可以通过ng-if来直接实现标签是否构建

<div ng-if="entity.goods.isEnableSpec==1">

-----------------------

</div>

$localtion.search =========>search 属性是一个可读可写的字符串，可设置或返回当前 URL 的查询部分（问号 ? 之后的部分）。

　`$location`服务负责解析浏览器地址栏中的URL（基于[window.location](https://developer.mozilla.org/en/window.location)），以便你的应用可以访问它。 这是一个双向同步机制 —— 对地址栏URL的任何修改都会被映射到$location服务中，对$location的任何修改也同样会被映射到地址栏。 

同时提供getter和setter方法。

```
// 获得当前path
$location.path();
// 修改当前path
$location.path('/newValue')
```

所有的setter方法都返回同一个`$location`对象，以便你进行链式调用。例如，要想一次性修改地址中的几个字段，可以用这样的语法把几个setter串起来：

  **在AngularJS中,如果在controller中用location需要一个注入服务操作**

```
$location.path('/newValue').search({key: value});
```

----

在angularJS中,如果在controller和service等js环境范围内中写scope变量必须加scope,而在body中需要写{{variable}},在ng的angularjs方法中则可以直接使用变量名在赋值



我们测试后发现高亮显示的html代码原样输出，这是angularJS为了防止html攻击采取的安全机制。我们如何在页面上显示html的结果呢？我们会用到$sce服务的trustAsHtml方法来实现转换。

```
app.filter('trustHtml',['$sce',function($sce){
    return function(data){
        return $sce.trustAsHtml(data);
    }
}]);
```

```
<div class="attr" ng-bind-html="item.title | trustHtml"></div>
```

复选框勾选进行赋值

```
<input type="checkbox"  ng-model="entity.goods.isEnableSpec" ng-true-value="1" ng-false-value="0">
```

#### $interval服务简介

 在AngularJS中$interval服务用来处理间歇性处理一些事情

```
 $interval(执行的函数,间隔的毫秒数,运行次数);
```

运行次数可以缺省，如果缺省则无限循环执行  

取消执行用cancel方法

```
$interval.cancel(time);
```

