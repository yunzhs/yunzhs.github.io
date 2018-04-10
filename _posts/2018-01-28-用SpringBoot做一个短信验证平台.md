---
layout:     post
title:      基于SpringBoot的一个短信验证小Demo
date:       2018-1-28
author:     yunzhs
header-img: img/Archer.jpg
catalog: true
tags:
        - SpringBoot
        - 业务操作
typora-root-url: ..
typora-copy-images-to: ..\img\posts
---

## 一.快速建立springboot项目

通过**Spring**官方提供的**Spring** **Initializr** 来构建Maven项目，它不仅完美支持IDEA和Eclipse，而且能自动生成**启动类和单元测试代码**，给开发人员带来极大的便利！！

可以在 [**Spring** **Initializr**](http://start.spring.io/) 中选择想要的配置,网站会自动生成一个项目zip,下载解压后即可直接导入到IDE中

![1517732879234](/img/posts/1517732879234.png)

​ idea 中直接提供了这种创建方式                                                                                                                                                                                                                                                                                                                

![1517734399054](/img/posts/1517734399054.png)

![1517734479739](/img/posts/1517734479739.png)

![Snipaste_2018-02-04_16-55-26](/img/posts/Snipaste_2018-02-04_16-55-26.png)

可以看到列表中详细支持,cloud中也有各种集群的配置,只想说springboot也很可能是未来的趋势,的确是让spring成为一个完全的轻量级的框架.

emmmmmm,1M的小水管下了五分钟的jar包,下载完毕后,即创建成功



## 2.短信发送平台-阿里大于

### 1.需要在阿里云上进行一些认证处理,充值的工作.

1.开通短信服务

2.申请签名,也就是短信前面的标志,比如说【工商银行]】就是签名

3.申请模板，按照规范，文明的方式进行书写，并且写一个{code}作为验证码

4.创建accessKeyu

5.充值，才能变得更强

### 2.SDK安装

从阿里云通信官网上下载Demo工程

把 `aliyun-java-sdk-core`  `alicom-dysms-api`  maven install 安装到本地仓库

```
<dependencies>
    	<dependency>
    		<groupId>com.aliyun</groupId>
    		<artifactId>aliyun-java-sdk-dysmsapi</artifactId>
    		<version>1.0.0-SNAPSHOT</version>
    	</dependency>
    	<dependency>
    		<groupId>com.aliyun</groupId>
    		<artifactId>aliyun-java-sdk-core</artifactId>
    		<version>3.2.5</version>
    	</dependency>
</dependencies>
```

将以上依赖导入到maven工程中

### 3.发送短信测试

1.导入阿里大于的SDK中的 alicom-dysms-api工程

2.填换自己的AK

![1517737861042](/img/posts/1517737861042.png)

3.填写需要的request内容

![1517737987032](/img/posts/1517737987032.png)

4.执行main方法，即可在上述填写的手机中得到你想要的短信

> ```
> SMS_123670964
> 雲中手
> LTAIoZgho1V3Fja6
> pkn1sEXIVjGIaOAjC1XaTp2GbR1U7o
> ```

### 4.短信微服务

配置文件

```
server.port=9003
spring.activemq.broker-url=tcp://服务器:61616
accessKeyId=不告诉你
accessKeySecret=不告诉你
```



短信工具类

```
@Component
public class SmsDemo {

    //产品名称:云通信短信API产品,开发者无需替换
    static final String product = "Dysmsapi";
    //产品域名,开发者无需替换
    static final String domain = "dysmsapi.aliyuncs.com";

    @Autowired
    private Environment env;



    public  SendSmsResponse sendSms(String mobile,String template_code,String sign_name,String param) throws ClientException {

        //可自助调整超时时间
        System.setProperty("sun.net.client.defaultConnectTimeout", "10000");
        System.setProperty("sun.net.client.defaultReadTimeout", "10000");

        //初始化acsClient,暂不支持region化
        IClientProfile profile = DefaultProfile.getProfile("cn-hangzhou", env.getProperty("accessKeyId"),  env.getProperty("accessKeySecret"));
        DefaultProfile.addEndpoint("cn-hangzhou", "cn-hangzhou", product, domain);
        IAcsClient acsClient = new DefaultAcsClient(profile);

        //组装请求对象-具体描述见控制台-文档部分内容
        SendSmsRequest request = new SendSmsRequest();
        //必填:待发送手机号
        request.setPhoneNumbers(mobile);
        //必填:短信签名-可在短信控制台中找到
        request.setSignName(sign_name);
        //必填:短信模板-可在短信控制台中找到
        request.setTemplateCode(template_code);
        //可选:模板中的变量替换JSON串,如模板内容为"亲爱的${name},您的验证码为${code}"时,此处的值为
        request.setTemplateParam(param);

        //选填-上行短信扩展码(无特殊需求用户请忽略此字段)
        //request.setSmsUpExtendCode("90997");

        //可选:outId为提供给业务方扩展字段,最终在短信回执消息中将此值带回给调用者
        request.setOutId("yourOutId");

        //hint 此处可能会抛出异常，注意catch
        SendSmsResponse sendSmsResponse = acsClient.getAcsResponse(request);

        return sendSmsResponse;
    }



}
```





监听类

```
 @Component
    public class SmsListener {
        @Autowired
        private SmsDemo smsUtil;

        @JmsListener(destination = "sms")
        public void sendSms(Map<String, String> map) {
            try {
                SendSmsResponse response = smsUtil.sendSms(
                        map.get("mobile"),
                        map.get("template_code"),
                        map.get("sign_name"),
                        map.get("param"));
                System.out.println("Code=" + response.getCode());
                System.out.println("Message=" + response.getMessage());
                System.out.println("RequestId=" + response.getRequestId());
                System.out.println("BizId=" + response.getBizId());
            } catch (ClientException e) {
                e.printStackTrace();
            }
        }

    }
```

测试方法,在controller类中实现

```
@Autowired
private JmsMessagingTemplate jmsMessagingTemplate;

@RequestMapping("/sendsms")
public  void sendSms(){
    Map map=new HashMap<>();
    map.put("mobile", "13900001111");
    map.put("template_code", "SMS_85735065");
    map.put("sign_name", "~~");
    map.put("param", "{\"number\":\"102931\"}");
    jmsMessagingTemplate.convertAndSend("sms",map);
}
```

### 完成

