---
layout:     post
title:      SpringBoot整合Swagger2
date:       2018-06-01
author:     yunzhs
header-img: img/Fate of Princess.jpg
catalog: true
tags:
    - SpringBoot
typora-root-url: ..
typora-copy-images-to: ..\img\posts
---

## 前言:

​       作为程序员公司写API文档数量应该不少，当然如果你还处在自己一个人开发前后台的年代，当我没说，如今为了前后台更好的对接，还是为了以后交接方便，都有要求写API文档。 

手写Api文档的几个痛点：

1. 文档需要更新的时候，需要再次发送一份给前端，也就是文档更新交流不及时。
2. 接口返回结果不明确
3. 不能直接在线测试接口，通常需要使用工具，比如postman
4. 接口文档太多，不好管理

Swagger也就是为了解决这个问题，当然也不能说Swagger就一定是完美的，当然也有缺点，最明显的就是代码移入性比较强。

其他的不多说，想要了解Swagger的，可以去Swagger官网，可以直接使用Swagger editor编写接口文档，当然我们这里讲解的是SpringBoot整合Swagger2，直接生成接口文档的方式。

## 一、依赖

```
<dependency>
   <groupId>io.springfox</groupId>
   <artifactId>springfox-swagger2</artifactId>
   <version>2.8.0</version>
</dependency>

<dependency>
   <groupId>io.springfox</groupId>
   <artifactId>springfox-swagger-ui</artifactId>
   <version>2.8.0</version>
</dependency>
```

## 二、Swagger配置类

其实这个配置类，只要了解具体能配置哪些东西就好了，毕竟这个东西配置一次之后就不用再动了。 特别要注意的是里面配置了api文件也就是controller包的路径，不然生成的文档扫描不到接口。

```
@Configuration
public class SwaggerConfig {

    @Bean
    public Docket createRestApi() {
        return new Docket(DocumentationType.SWAGGER_2)
                .apiInfo(apiInfo())
                .select()
                .apis(RequestHandlerSelectors.basePackage("com.transinfo.waterage.controller"))
                .paths(PathSelectors.any())
                .build();
    }

    private ApiInfo apiInfo() {
        return new ApiInfoBuilder()
                .title("水运前端api文档")
                .description("水运前端API文档")
                .termsOfServiceUrl("")
                .version("1.0")
                .build();
    }
}
```

Application.class 加上注解@EnableSwagger2 表示开启Swagger

```
package cn.saytime;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import springfox.documentation.swagger2.annotations.EnableSwagger2;

@SpringBootApplication
@EnableSwagger2
public class SpringbootSwagger2Application {

	public static void main(String[] args) {
		SpringApplication.run(SpringbootSwagger2Application.class, args);
	}
}
```

### 三、注解操作

在接口方法中添加ApiOperation注解,有value和note两种属性,都是起注解说明的作用

```
@ApiOperation(value = "查询")
@RequestMapping(value = "/ccc" ,method = RequestMethod.POST, produces = BaseConst.UTF_8_APPLICATION_JSON)
@ResponseBody
public CarRS findCar(HttpServletRequest request,@RequestBody UserCarDTO userCarDTO){
    HttpSession session = request.getSession();
    String response = carService.findCarService(userCarDTO.getUserId());
    CarRS carRS=JsonTool.INSTANCE.jsonStrToObj(response,CarRS.class);
    return carRS;
}
```

 在请求和返回对应的实体类分别添加注解:

```
@ApiModel
public class FrontLoginRQ {


    private PublicRequest publicRequest;
    @ApiModelProperty(value = "手机号",required = true)
    private String mobile;
    @ApiModelProperty(value = "密码",required = true)
    private String password;
    @ApiModelProperty(value = "图片验证码",required = true)
    private String code;
```

还有一个allowEmptyValue参数表示是否能为空,默认是不能为空的.

传入是固定参数,则可以用这种方法:ApiImplicitParams

```
	@ApiOperation(value="更新信息", notes="根据url的id来指定更新用户信息")
	@ApiImplicitParams({
			@ApiImplicitParam(name = "id", value = "用户ID", required = true, dataType = "Long",paramType = "path"),
			@ApiImplicitParam(name = "user", value = "用户实体user", required = true, dataType = "User")
	})
	@RequestMapping(value = "user/{id}", method = RequestMethod.PUT)
	public ResponseEntity<JsonResult> update (@PathVariable("id") Integer id, @RequestBody User user){
		JsonResult r = new JsonResult();
```

![1527832969263](/img/posts/1527832969263.png)

![1527833047016](/img/posts/1527833047016.png)