---
layout:     post
title:      bootstrapTable知识点整理
date:       2018-09-25
author:     yunzhs
header-img: img/post-bg-rwd.jpg
catalog: true
tags:
    - 前端
typora-root-url: ..
typora-copy-images-to: ..\img\posts
---

### 官方举例

![1537856195386](/img/posts/1537856195386.png)

### 实现表格中按钮响应

![1537856514745](/img/posts/1537856514745.png)

官方源码

![1537837837029](/img/posts/1537837837029.png)

我的代码:

```
{title: '操作', align: 'center', valign: 'middle',events:operateEvents,formatter: operateFormatter,width:"150px"}
```

```
function operateFormatter(value, row, index) {
    OrderInfo.seItem=row;
    if(OrderInfo.seItem.confirmStatus!=='已确认'){
        return [
            '<button id="mmmmmm" type="button" class="RoleOfedit  btn btn-success ">确认</button>&nbsp;&nbsp;',
            '<button id="mmmmmm1" type="button" class="RoleOfedit1  btn btn-info">详情</button>',
        ].join('')
    }
    else {
        return [
            '<button id="mmmmmm" type="button" class="nonono  btn btn-default ">已确认</button>&nbsp;&nbsp;',
            '<button id="mmmmmm1" type="button" class="RoleOfedit1  btn btn-info">详情</button>',
        ].join('')
    }

}
window.operateEvents = {
    'click .RoleOfedit': function (e, value, row, index) {
        OrderInfo.seItem=row;
        OrderInfo.editSubmit();
    },
    'click .RoleOfedit1': function (e, value, row, index) {
        OrderInfo.seItem=row;
        OrderInfo.openOrderInfoDetail()
    }
};
```

### 去掉表头和表体的多余内容省略功能

以guns的表格封装为例

默认:

```
td{  
     width:100%;  
     word-break:keep-all;/* 不换行 */  
     white-space:nowrap;/* 不换行 */  
     overflow:hidden;/* 内容超出宽度时隐藏超出部分的内容 */  
     text-overflow:ellipsis;/* 当对象内文本溢出时显示省略标记(...) ；需与overflow:hidden;一起使用*/  
}
```

修改:

```
th,td{
    width:100%;
    word-break:normal;/* 不换行 */
    white-space: normal;
    overflow: auto;/* 内容超出宽度时隐藏超出部分的内容 */
    /*text-overflow:ellipsis;!* 当对象内文本溢出时显示省略标记(...) ；需与overflow:hidden;一起使用*!*/
}
```

修改static/css/plugins/bootstrap-treetable/bootstrap-treetable.css

添加以下代码

```
.scs {
    width: 200px;
    font-size: 12px;
    border: 1px solid #ddd;
    overflow: hidden;
    text-overflow: ellipsis;
    white-space: nowrap;
}
```

修改static/js/plugins/bootstrap-treetable/bootstrap-treetable.js

![1537858977430](/img/posts/1537858977430.png)