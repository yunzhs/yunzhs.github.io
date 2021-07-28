---
layout:     post
title:      通过实现comparable来实现对象在List中的排序
date:       2018-07-19
author:     yunzhs
header-img: img/post-bg-rwd.jpg
catalog: true
tags:
    - 随笔
typora-root-url: ..
typora-copy-images-to: ..\img\posts
---

在调用一些接口时,往往回返回一些值,而这些值我们无法去改变它的顺序,因此需要我们去实现一些对象的排序,因此可以通过以下的方式来进行.

实现comparable接口

重写compareTo方法,用compare方法重写要比较的参数

实例:

```
public class UserCar extends IdEntity implements Comparable<UserCar>{
    @ApiModelProperty(value = "前台用户信息")
    private SysFrontUser sysFrontUser;
    @ApiModelProperty(value = "车辆ID")
    private String carNo;
    @ApiModelProperty(value = "汽车类型")
    private String carType;
    @ApiModelProperty(value = "车身长度")
    private BigDecimal truckLength;
    @ApiModelProperty(value = "车身高度")
    private BigDecimal truckHeight;
    @ApiModelProperty(value = "车身宽度")
    private BigDecimal truckWide;
    @ApiModelProperty(value = "司机姓名")
    private String driverName;
    @ApiModelProperty(value = "司机身份证类型")
    private String driverIdType;
    @ApiModelProperty(value = "司机身份证号")
    private String driverIdNo;
    @ApiModelProperty(value = "司机手机")
    private String driverMobile;
    
    '''''''''''''''''''''''''''''''''''''''''''''''
        @Override
    public int compareTo(UserCar userCar) {
        long hight1=this.getTruckHeight();
        long hight2=userCar.getTruckHeight();
        return (Long.compare(hight1, hight2));
    }
}
```

调用方法:

```
-------------
此处定义三个usercar对象
-------------


List<UserCar> UserCars=new ArrayList<UserCar>();
UserCars.add(u1);
UserCars.add(u2);
UserCars.add(u3);
Collections.sort(UserCars);


-------------
然后就会发现UserCars已经是一个按照升序排序的集合
-------------
```

