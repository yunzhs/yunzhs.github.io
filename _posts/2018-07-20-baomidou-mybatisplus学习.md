---
layout:     post
title:      baomidou-mybatisplus学习
date:       2018-07-20
author:     yunzhs
header-img: img/post-bg-rwd.jpg
catalog: true
tags:
    - mybatis
typora-root-url: ..
typora-copy-images-to: ..\img\posts
---

## 一.前言

最近在写一个后台项目,发现有很多以前没用过的新的东西,于是整理一下相关的知识,其中涉及了许多mybatis的东西,上个项目用的是hibernate加JPA的dao处理模式,一度觉得JPA比普通的mybatis好用了不少,在接触这个项目后,发现mybatis的相关插件也有很多先进的东西,于是找到项目所用的mybatis框架,从头学习一下官方文档.

[Mybatis-Plus](https://github.com/baomidou/mybatis-plus)（简称MP）是一个 [Mybatis](http://www.mybatis.org/mybatis-3/) 的增强工具，在 Mybatis 的基础上只做增强不做改变，为简化开发、提高效率而生。

> 我们的愿景是成为 `Mybatis` 最好的搭档，就像 [魂斗罗](https://raw.githubusercontent.com/baomidou/mybatis-plus-doc/master/assets/contra.jpg) 中的1P、2P，基友搭配，效率翻倍。

![relationship](http://mp.baomidou.com/assets/relationship-with-mybatis.png)

### 特性

- **无侵入**：Mybatis-Plus 在 Mybatis 的基础上进行扩展，只做增强不做改变，引入 Mybatis-Plus 不会对您现有的 Mybatis 构架产生任何影响，而且 MP 支持所有 Mybatis 原生的特性
- **依赖少**：仅仅依赖 Mybatis 以及 Mybatis-Spring
- **损耗小**：启动即会自动注入基本 CURD，性能基本无损耗，直接面向对象操作
- **预防Sql注入**：内置 Sql 注入剥离器，有效预防Sql注入攻击
- **通用CRUD操作**：内置通用 Mapper、通用 Service，仅仅通过少量配置即可实现单表大部分 CRUD 操作，更有强大的条件构造器，满足各类使用需求
- **多种主键策略**：支持多达4种主键策略（内含分布式唯一ID生成器），可自由配置，完美解决主键问题
- **支持热加载**：Mapper 对应的 XML 支持热加载，对于简单的 CRUD 操作，甚至可以无 XML 启动
- **支持ActiveRecord**：支持 ActiveRecord 形式调用，实体类只需继承 Model 类即可实现基本 CRUD 操作
- **支持代码生成**：采用代码或者 Maven 插件可快速生成 Mapper 、 Model 、 Service 、 Controller 层代码，支持模板引擎，更有超多自定义配置等您来使用（P.S. 比 Mybatis 官方的 Generator 更加强大！）
- **支持自定义全局通用操作**：支持全局通用方法注入（ Write once, use anywhere ）
- **支持关键词自动转义**：支持数据库关键词（order、key......）自动转义，还可自定义关键词
- **内置分页插件**：基于 Mybatis 物理分页，开发者无需关心具体操作，配置好插件之后，写分页等同于普通List查询
- **内置性能分析插件**：可输出 Sql 语句以及其执行时间，建议开发测试时启用该功能，能有效解决慢查询
- **内置全局拦截插件**：提供全表 delete 、 update 操作智能分析阻断，预防误操作

## 二.简单示例(ActiveRecord)

ActiveRecord 一直广受动态语言（ PHP 、 Ruby 等）的喜爱，而 Java 作为准静态语言，对于 ActiveRecord 往往只能感叹其优雅，所以我们也在 AR 道路上进行了一定的探索，喜欢大家能够喜欢，也同时欢迎大家反馈意见与建议。

我们如何使用 AR 模式？

```
@TableName("sys_user") // 注解指定表名
public class User extends Model<User> {

  ... // fields

  ... // getter and setter

  /** 指定主键 */
  @Override
  protected Serializable pkVal() {
      return this.id;
  }
}
```

我们仅仅需要继承 Model 类且实现主键指定方法 即可让实体开启 AR 之旅，开启 AR 之路后，我们如何使用它呢？

> 基本CRUD

```
// 初始化 成功标识
boolean result = false;
// 初始化 User
User user = new User();

// 保存 User
user.setName("Tom");
result = user.insert();

// 更新 User
user.setAge(18);
result = user.updateById();

// 查询 User
User exampleUser = t1.selectById();

// 查询姓名为‘张三’的所有用户记录
List<User> userList1 = user.selectList(
        new EntityWrapper<User>().eq("name", "张三")
);

// 删除 User
result = t2.deleteById();
```

> 分页操作

```
// 分页查询 10 条姓名为‘张三’的用户记录
List<User> userList = user.selectPage(
        new Page<User>(1, 10),
        new EntityWrapper<User>().eq("name", "张三")
).getRecords();
```

> 复杂操作

```
// 分页查询 10 条姓名为‘张三’、性别为男，且年龄在18至50之间的用户记录
List<User> userList = user.selectPage(
        new Page<User>(1, 10),
        new EntityWrapper<User>().eq("name", "张三")
                .eq("sex", 0)
                .between("age", "18", "50")
).getRecords();
```

## 三.SpringBoot的安装集成

spring boot 项目集成mp可以使用starter

- pom.xml引入

```xml
<dependencies>
  <dependency>
      <groupId>com.baomidou</groupId>
      <artifactId>mybatis-plus-boot-starter</artifactId>
      <version>最新版本号</version>
  </dependency>
</dependencies>
<!-- 如果mapper.xml是放在src/main/java目录下，需配置以下-->
<build>
  <resources>
      <resource>
          <directory>src/main/java</directory>
          <filtering>false</filtering>
          <includes>
              <include>**/mapper/*.xml</include>
          </includes>
      </resource>
  </resources>
</build>
```

application.yml配置文件

```yaml
mybatis-plus:
  # 如果是放在src/main/java目录下 classpath:/com/yourpackage/*/mapper/*Mapper.xml
  # 如果是放在resource目录 classpath:/mapper/*Mapper.xml
  mapper-locations: classpath:/mapper/*Mapper.xml
  #实体扫描，多个package用逗号或者分号分隔
  typeAliasesPackage: com.yourpackage.*.entity
  global-config:
    #主键类型  0:"数据库ID自增", 1:"用户输入ID",2:"全局唯一ID (数字类型唯一ID)", 3:"全局唯一ID UUID";
    id-type: 3
    #字段策略 0:"忽略判断",1:"非 NULL 判断"),2:"非空判断"
    field-strategy: 2
    #驼峰下划线转换
    db-column-underline: true
    #mp2.3+ 全局表前缀 mp_
    #table-prefix: mp_
    #刷新mapper 调试神器
    #refresh-mapper: true
    #数据库大写下划线转换
    #capital-mode: true
    # Sequence序列接口实现类配置
    key-generator: com.baomidou.mybatisplus.incrementer.OracleKeyGenerator
    #逻辑删除配置（下面3个配置）
    logic-delete-value: 1
    logic-not-delete-value: 0
    sql-injector: com.baomidou.mybatisplus.mapper.LogicSqlInjector
    #自定义填充策略接口实现
    meta-object-handler: com.baomidou.springboot.MyMetaObjectHandler
  configuration:
    #配置返回数据库(column下划线命名&&返回java实体是驼峰命名)，自动匹配无需as（没开启这个，SQL需要写as： select user_id as userId） 
    map-underscore-to-camel-case: true
    cache-enabled: false
    #配置JdbcTypeForNull, oracle数据库必须配置
    jdbc-type-for-null: 'null'
```

Java Configuration配置[更多配置参考](https://gitee.com/baomidou/mybatisplus-spring-boot/blob/master/src/main/java/com/baomidou/springboot/config/MybatisPlusConfig.java)

```
@Configuration
@MapperScan("com.yourpackage.*.mapper*")
public class MybatisPlusConfig {
   /*
    * 分页插件，自动识别数据库类型
    * 多租户，请参考官网【插件扩展】
    */
   @Bean
   public PaginationInterceptor paginationInterceptor() {
      return new PaginationInterceptor();
   }

   /*
    * oracle数据库配置JdbcTypeForNull
    * 参考：https://gitee.com/baomidou/mybatisplus-boot-starter/issues/IHS8X
    不需要这样配置了，参考 yml:
    mybatis-plus:
      confuguration
        dbc-type-for-null: 'null' 
   @Bean
   public ConfigurationCustomizer configurationCustomizer(){
       return new MybatisPlusCustomizers();
   }

   class MybatisPlusCustomizers implements ConfigurationCustomizer {

       @Override
       public void customize(org.apache.ibatis.session.Configuration configuration) {
           configuration.setJdbcTypeForNull(JdbcType.NULL);
       }
   }
   */
}
```
## 四.代码生成器

在代码生成之前，首先进行配置，MP提供了大量的自定义设置，生成的代码完全能够满足各类型的需求 

默认配置

```
import org.junit.Test;

import com.baomidou.mybatisplus.generator.AutoGenerator;
import com.baomidou.mybatisplus.generator.config.DataSourceConfig;
import com.baomidou.mybatisplus.generator.config.GlobalConfig;
import com.baomidou.mybatisplus.generator.config.PackageConfig;
import com.baomidou.mybatisplus.generator.config.StrategyConfig;
import com.baomidou.mybatisplus.generator.config.rules.DbType;
import com.baomidou.mybatisplus.generator.config.rules.NamingStrategy;

/**
 * <p>
 * 测试生成代码
 * </p>
 *
 * @author K神
 * @date 2017/12/18
 */
public class GeneratorServiceEntity {

    @Test
    public void generateCode() {
        String packageName = "com.baomidou.springboot";
        boolean serviceNameStartWithI = false;//user -> UserService, 设置成true: user -> IUserService
        generateByTables(serviceNameStartWithI, packageName, "user", "role");
    }

    private void generateByTables(boolean serviceNameStartWithI, String packageName, String... tableNames) {
        GlobalConfig config = new GlobalConfig();
        String dbUrl = "jdbc:mysql://localhost:3306/mybatis-plus";
        DataSourceConfig dataSourceConfig = new DataSourceConfig();
        dataSourceConfig.setDbType(DbType.MYSQL)
                .setUrl(dbUrl)
                .setUsername("root")
                .setPassword("")
                .setDriverName("com.mysql.jdbc.Driver");
        StrategyConfig strategyConfig = new StrategyConfig();
        strategyConfig
                .setCapitalMode(true)
                .setEntityLombokModel(false)
                .setDbColumnUnderline(true)
                .setNaming(NamingStrategy.underline_to_camel)
                .setInclude(tableNames);//修改替换成你需要的表名，多个表名传数组
        config.setActiveRecord(false)
                .setAuthor("K神带你飞")
                .setOutputDir("d:\\codeGen")
                .setFileOverride(true);
        if (!serviceNameStartWithI) {
            config.setServiceName("%sService");
        }
        new AutoGenerator().setGlobalConfig(config)
                .setDataSource(dataSourceConfig)
                .setStrategy(strategyConfig)
                .setPackageInfo(
                        new PackageConfig()
                                .setParent(packageName)
                                .setController("controller")
                                .setEntity("entity")
                ).execute();
    }

    private void generateByTables(String packageName, String... tableNames) {
        generateByTables(true, packageName, tableNames);
    }
}
```

## 五.条件构造器

- 翻页查询

```
public Page<T> selectPage(Page<T> page, EntityWrapper<T> entityWrapper) {
  if (null != entityWrapper) {
      entityWrapper.orderBy(page.getOrderByField(), page.isAsc());
  }
  page.setRecords(baseMapper.selectPage(page, entityWrapper));
  return page;
}
```

- 拼接 sql 方式 一

```
@Test
public void testTSQL11() {
    /*
     * 实体带查询使用方法  输出看结果
     */
    EntityWrapper<User> ew = new EntityWrapper<User>();
    ew.setEntity(new User(1));
    ew.where("user_name={0}", "'zhangsan'").and("id=1")
            .orNew("user_status={0}", "0").or("status=1")
            .notLike("user_nickname", "notvalue")
            .andNew("new=xx").like("hhh", "ddd")
            .andNew("pwd=11").isNotNull("n1,n2").isNull("n3")
            .groupBy("x1").groupBy("x2,x3")
            .having("x1=11").having("x3=433")
            .orderBy("dd").orderBy("d1,d2");
    System.out.println(ew.getSqlSegment());
}
```

- 拼接 sql 方法二

```
int buyCount = selectCount(Condition.create()
                .setSqlSelect("sum(quantity)")
                .isNull("order_id")
                .eq("user_id", 1)
                .eq("type", 1)
                .in("status", new Integer[]{0, 1})
                .eq("product_id", 1)
                .between("created_time", startDate, currentDate)
                .eq("weal", 1));
```

 	此方法直接在impl层实现即可,不需到mapper层进行写入,由MP进行封装查询.

​	 Condition（与EW类似） 来让用户自由的构建查询条件，简单便捷，没有额外的负担，能够有效提高开发效率。 

一个合适的实现类方式

```
@Service
public class OrderInfoServiceImpl extends ServiceImpl<OrderInfoMapper, OrderInfo> implements IOrderInfoService {
```



- 自定义 SQL 方法如何使用 Wrapper

mapper java 接口方法

```
List<User> selectMyPage(RowBounds rowBounds, @Param("ew") Wrapper<T> wrapper);
```

mapper xml 定义

```
<select id="selectMyPage" resultType="User">
  SELECT * FROM user 
  <where>
  ${ew.sqlSegment}
  </where>
</select>
```

### 条件参数说明

| 查询方式     | 说明                              |
| ------------ | --------------------------------- |
| setSqlSelect | 设置 SELECT 查询字段              |
| where        | WHERE 语句，拼接 + `WHERE 条件`   |
| and          | AND 语句，拼接 + `AND 字段=值`    |
| andNew       | AND 语句，拼接 + `AND (字段=值)`  |
| or           | OR 语句，拼接 + `OR 字段=值`      |
| orNew        | OR 语句，拼接 + `OR (字段=值)`    |
| eq           | 等于=                             |
| allEq        | 基于 map 内容等于=                |
| ne           | 不等于<>                          |
| gt           | 大于>                             |
| ge           | 大于等于>=                        |
| lt           | 小于<                             |
| le           | 小于等于<=                        |
| like         | 模糊查询 LIKE                     |
| notLike      | 模糊查询 NOT LIKE                 |
| in           | IN 查询                           |
| notIn        | NOT IN 查询                       |
| isNull       | NULL 值查询                       |
| isNotNull    | IS NOT NULL                       |
| groupBy      | 分组 GROUP BY                     |
| having       | HAVING 关键词                     |
| orderBy      | 排序 ORDER BY                     |
| orderAsc     | ASC 排序 ORDER BY                 |
| orderDesc    | DESC 排序 ORDER BY                |
| exists       | EXISTS 条件语句                   |
| notExists    | NOT EXISTS 条件语句               |
| between      | BETWEEN 条件语句                  |
| notBetween   | NOT BETWEEN 条件语句              |
| addFilter    | 自由拼接 SQL                      |
| last         | 拼接在最后，例如：last("LIMIT 1") |

## 六.分页

- UserMapper.java 方法内容

```
public interface UserMapper{//可以继承或者不继承BaseMapper
    /**
     * <p>
     * 查询 : 根据state状态查询用户列表，分页显示
     * </p>
     *
     * @param page
     *            翻页对象，可以作为 xml 参数直接使用，传递参数 Page 即自动分页
     * @param state
     *            状态
     * @return
     */
    List<User> selectUserList(Pagination page, Integer state);
}
```

- UserServiceImpl.java 调用翻页方法，需要 page.setRecords 回传给页面

```
public Page<User> selectUserPage(Page<User> page, Integer state) {
    // 不进行 count sql 优化，解决 MP 无法自动优化 SQL 问题
    // page.setOptimizeCountSql(false);
    // 不查询总记录数
    // page.setSearchCount(false);
    // 注意！！ 分页 total 是经过插件自动 回写 到传入 page 对象
    return page.setRecords(userMapper.selectUserList(page, state));
}
```

- UserMapper.xml 等同于编写一个普通 list 查询，mybatis-plus 自动替你分页

```
<select id="selectUserList" resultType="User">
    SELECT * FROM user WHERE state=#{state}
</select>
```

- 一个获取page属性的工具类,需要存在session里取

```
public class PageFactory<T> {

    public Page<T> defaultPage() {
        HttpServletRequest request = HttpKit.getRequest();
        int limit = Integer.valueOf(request.getParameter("limit"));     //每页多少条数据
        int offset = Integer.valueOf(request.getParameter("offset"));   //每页的偏移量(本页当前有多少条)
        String sort = request.getParameter("sort");         //排序字段名称
        String order = request.getParameter("order");       //asc或desc(升序或降序)
        if (ToolUtil.isEmpty(sort)) {
            Page<T> page = new Page<>((offset / limit + 1), limit);
            page.setOpenSort(false);
            return page;
        } else {
            Page<T> page = new Page<>((offset / limit + 1), limit, sort);
            if (Order.ASC.getDes().equals(order)) {
                page.setAsc(true);
            } else {
                page.setAsc(false);
            }
            return page;
        }
    }
}
```

controller

```
        Page<OperationLog> page = new PageFactory<OperationLog>().defaultPage();
```

