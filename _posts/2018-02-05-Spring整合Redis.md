---
layout:     post
title:      Spring整合Redis
subtitle:   
date:       2017-09-25
author:     yunzhs
header-img: img/SMILE.jpg
catalog: true
tags:
    - spring
    - Redis
typora-root-url: ..
typora-copy-images-to: ..\img\posts
---

## 一.SpirngRedis简介

Spring-data-redis是spring大家族的一部分，提供了在srping应用中通过简单的配置访问redis服务，对reids底层开发包(Jedis,  JRedis, and RJC)进行了高度封装，RedisTemplate提供了redis各种操作、异常处理及序列化，支持发布订阅，并对spring 3.1 cache进行了实现。

spring-data-redis针对jedis提供了如下功能：
​         1.连接池自动管理，提供了一个高度封装的“RedisTemplate”类
​         2.针对jedis客户端中大量api进行了归类封装,将同一类型操作封装为operation接口
​         ValueOperations：简单K-V操作
​         SetOperations：set类型数据操作
​         ZSetOperations：zset类型数据操作
​         HashOperations：针对map类型的数据操作
​         ListOperations：针对list类型的数据操作

##二.入门demo

###所需的相关依赖

```
<!-- 缓存 -->
<dependency> 
		  <groupId>redis.clients</groupId> 
		  <artifactId>jedis</artifactId> 
		  <version>2.8.1</version> 
</dependency> 
<dependency> 
		  <groupId>org.springframework.data</groupId> 
		  <artifactId>spring-data-redis</artifactId> 
		  <version>1.7.2.RELEASE</version> 
</dependency>	

```

### 配置文件

src/main/resources下创建properties文件夹，建立redis-config.properties ,根据自己的需求进行配置

```
redis.host=127.0.0.1
redis.port=6379
redis.pass=
redis.database=0
redis.maxIdle=300
redis.maxWait=3000
redis.testOnBorrow=true
```

在src/main/resources下创建spring文件夹 ，创建applicationContext-redis.xml

```
   <context:property-placeholder location="classpath*:properties/*.properties" />   
   <!-- redis 相关配置 --> 
   <bean id="poolConfig" class="redis.clients.jedis.JedisPoolConfig">  
     <property name="maxIdle" value="${redis.maxIdle}" />   
     <property name="maxWaitMillis" value="${redis.maxWait}" />  
     <property name="testOnBorrow" value="${redis.testOnBorrow}" />  
   </bean>  
   <bean id="JedisConnectionFactory" class="org.springframework.data.redis.connection.jedis.JedisConnectionFactory" 
       p:host-name="${redis.host}" p:port="${redis.port}" p:password="${redis.pass}" p:pool-config-ref="poolConfig"/>  
   
   <bean id="redisTemplate" class="org.springframework.data.redis.core.RedisTemplate">  
     <property name="connectionFactory" ref="JedisConnectionFactory" />  
   </bean>  

```

maxIdle ：最大空闲数

maxWaitMillis:连接时的最大等待毫秒数

testOnBorrow：在提取一个jedis实例时，是否提前进行验证操作；如果为true，则得到的jedis实例均是可用的；

### string值的操作

```
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations="classpath:spring/applicationContext-redis.xml")
public class TestValue {
	@Autowired
	private RedisTemplate redisTemplate;	
	@Test
	public void setValue(){
		redisTemplate.boundValueOps("name").set("itcast");		
	}	
	@Test
	public void getValue(){
		String str = (String) redisTemplate.boundValueOps("name").get();
		System.out.println(str);
	}	
	@Test
	public void deleteValue(){
		redisTemplate.delete("name");;
	}	
}

```

### Set类型操作

```
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations="classpath:spring/applicationContext-redis.xml")
public class TestSet {
	
	@Autowired
	private RedisTemplate redisTemplate;
	
	/**
	 * 存入值
	 */
	@Test
	public void setValue(){
		redisTemplate.boundSetOps("nameset").add("曹操");		
		redisTemplate.boundSetOps("nameset").add("刘备");	
		redisTemplate.boundSetOps("nameset").add("孙权");
	}
	
	/**
	 * 提取值
	 */
	@Test
	public void getValue(){
		Set members = redisTemplate.boundSetOps("nameset").members();
		System.out.println(members);
	}
	
	/**
	 * 删除集合中的某一个值
	 */
	@Test
	public void deleteValue(){
		redisTemplate.boundSetOps("nameset").remove("孙权");
	}
	
	/**
	 * 删除整个集合
	 */
	@Test
	public void deleteAllValue(){
		redisTemplate.delete("nameset");
	}
}

```

### List类型操作

####右压栈

```
/**
	 * 右压栈：后添加的对象排在后边
	 */
	@Test
	public void testSetValue1(){		
		redisTemplate.boundListOps("namelist1").rightPush("刘备");
		redisTemplate.boundListOps("namelist1").rightPush("关羽");
		redisTemplate.boundListOps("namelist1").rightPush("张飞");		
	}
	
	/**
	 * 显示右压栈集合
	 */
	@Test
	public void testGetValue1(){
		List list = redisTemplate.boundListOps("namelist1").range(0, 10);
		System.out.println(list);
	}

```

运行结果：

[刘备, 关羽, 张飞]

#### 左压栈

```
	/**
	 * 左压栈：后添加的对象排在前边
	 */
	@Test
	public void testSetValue2(){		
		redisTemplate.boundListOps("namelist2").leftPush("刘备");
		redisTemplate.boundListOps("namelist2").leftPush("关羽");
		redisTemplate.boundListOps("namelist2").leftPush("张飞");		
	}
	
	/**
	 * 显示左压栈集合
	 */
	@Test
	public void testGetValue2(){
		List list = redisTemplate.boundListOps("namelist2").range(0, 10);
		System.out.println(list);
	}

```

运行结果：

[张飞, 关羽, 刘备]

#### 根据索引查询元素

```
	/**
	 * 查询集合某个元素
	 */
	@Test
	public void testSearchByIndex(){
		String s = (String) redisTemplate.boundListOps("namelist1").index(1);
		System.out.println(s);
	}

```

### Hash类型操作

**存入值**

```
	@Test
	public void testSetValue(){
		redisTemplate.boundHashOps("namehash").put("a", "唐僧");
		redisTemplate.boundHashOps("namehash").put("b", "悟空");
		redisTemplate.boundHashOps("namehash").put("c", "八戒");
		redisTemplate.boundHashOps("namehash").put("d", "沙僧");
	}

```

**提取所有的KEY**

```
	@Test
	public void testGetKeys(){
		Set s = redisTemplate.boundHashOps("namehash").keys();		
		System.out.println(s);		
	}

```

**提取所有的值**

```
	@Test
	public void testGetValues(){
		List values = redisTemplate.boundHashOps("namehash").values();
		System.out.println(values);		
	}

```

**根据KEY提取值**

```
	@Test
	public void testGetValueByKey(){
		Object object = redisTemplate.boundHashOps("namehash").get("b");
		System.out.println(object);
	}

```

## 解决中文存入redis数据乱码的问题

在配置文件中添加stringRedisSerializer,并修改redisTemplate

```
<bean id="stringRedisSerializer" class="org.springframework.data.redis.serializer.StringRedisSerializer">
</bean>

<!-- Redis Template -->
<bean id="redisTemplate" class="org.springframework.data.redis.core.RedisTemplate">
        <property name="connectionFactory" ref="jedisConnectionFactory" />
        <!-- 新增 -->
        <property name="keySerializer" ref="stringRedisSerializer" />
        <property name="hashKeySerializer" ref="stringRedisSerializer" />
</bean>
```

