---
typora-root-url: ..
typora-copy-images-to: ..\img\posts
layout:     post
title:      Spring Data Solr学习
date:       2017-08-06
author:     yunzhs
header-img: img/Archer.jpg
catalog: true
tags:
    - spring
    - Solr
---

# Spring Data Solr

## 一.前言

将Solr的应用集成到Spring中,Spring Data Solr就是为了方便Solr的开发所研制的一个框架，其底层是对SolrJ（官方API）的封装。



### 二.配置文件

```
	<!-- solr服务器地址 -->
	<solr:solr-server id="solrServer" url="http://127.0.0.1:8080/solr" />
		
	<!-- solr模板，使用solr模板可对索引库进行CRUD的操作 -->
	<bean id="solrTemplate" class="org.springframework.data.solr.core.SolrTemplate">
		<constructor-arg ref="solrServer" />
	</bean>
```



## 三.基本使用

### @Field 注解

将pojo中的实体类的属性导入到solr中

如果属性与配置文件schema.xml域名称不一致，需要在注解中指定域名称



### 使用方法:

可以在相应的工具类中,创建SolrTemplate 的实例,然后对其进行操作

![1516524757351](/img/posts/1516524757351.png)

#### 常用的方法:

savebean();将bean实例保存到solrTemplate中

commit();将模板数据提交到solr中

**查询:**

```
public void testFindOne(){
		TbItem item = solrTemplate.getById(1, TbItem.class);
		System.out.println(item.getTitle());
	}
```

deleteById();按id进行删除

**分页查询:**

```
public void testPageQuery(){
		Query query=new SimpleQuery("*:*");//关键
		query.setOffset(20);//开始索引（默认0）
		query.setRows(20);//每页记录数(默认10)
		ScoredPage<TbItem> page = solrTemplate.queryForPage(query, TbItem.class);
		System.out.println("总记录数："+page.getTotalElements());
		List<TbItem> list = page.getContent();
		showList(list);
	}	
	//显示记录数据
	private void showList(List<TbItem> list){		
		for(TbItem item:list){
			System.out.println(item.getTitle() +item.getPrice());
		}		
	}
```

**条件查询:**

在query中添加criteria条件

```
		Criteria criteria=new Criteria("item_title").contains("2");
		criteria=criteria.and("item_title").contains("5");		
		query.addCriteria(criteria);
```

**删除所有数据:**

```
	Query query=new SimpleQuery("*:*");
	solrTemplate.delete(query);
		solrTemplate.commit();
```



## 四.实例操作

### 1.高亮显示

```

HighlightQuery query = new SimpleHighlightQuery();

//高亮设置
HighlightOptions highlightOptions=new HighlightOptions();
highlightOptions.addField("item_title");//对title字段进行高亮设置
highlightOptions.setSimplePrefix("<em style='color:red'>");//设置前缀
highlightOptions.setSimplePostfix("</em>");//设置后缀
query.setHighlightOptions(highlightOptions);

//根据关键字进行条件查询
Criteria criteria = new Criteria("item_keywords").is(searchMap.get("keywords"));
query.addCriteria(criteria);

```

然后获取高亮内容，设置到title字段上

这里虽然已经完成对高亮的添加,但并没有将高亮的内容赋给我们要获得的内容上

```
HighlightPage<TbItem> queryForHighlightPage = solrTemplate.queryForHighlightPage(query , TbItem.class);
		//获取高亮入口集合
		List<HighlightEntry<TbItem>> highlighted = queryForHighlightPage.getHighlighted();
		for(HighlightEntry<TbItem> h:highlighted){//对每一条数据进行遍历
			TbItem item = h.getEntity();//得到还没有高亮对象
			List<Highlight> highlights = h.getHighlights();//高亮对象集合 比如有多个列进行高亮，得到就是高亮字段的集合
			List<String> snipplets = highlights.get(0).getSnipplets();//获取片的集合,这里的片就是类似于spec规格的那种复杂值
			
			item.setTitle(snipplets.get(0));//获取高亮内容，设置到title字段上
		}
		List<TbItem> content = queryForHighlightPage.getContent(); 
```

### 2.添加过滤

```
FilterQuery filterQuery = new SimpleFilterQuery();
Criteria filterCriteria = new Criteria("item_category").is(searchMap.get("category"));
filterQuery.addCriteria(filterCriteria );
query.addFilterQuery(filterQuery );
```

上述代码表示添加了一个指定字段item_category为category的过滤,只有符合条件的才能被查询到

```
Criteria filterCriteria=new Criteria("item_price").greaterThanEqual(price[0]);
FilterQuery filterQuery=new SimpleFilterQuery(filterCriteria);
query.addFilterQuery(filterQuery);	
```

```
Criteria filterCriteria=new  Criteria("item_price").lessThanEqual(price[1]);
FilterQuery filterQuery=new SimpleFilterQuery(filterCriteria);
query.addFilterQuery(filterQuery);	
```

实现一个取大于price[0]和小于price[1]之间数

### 3.设置每页显示的数据数量和从哪条数据开始查询

==**这个设置主要用来做分页**==

```
 query.setOffset((pageNo-1)*pageSize);//从第几条记录查询
 query.setRows(pageSize);每页显示的数据数量
```

```
Page.getTotalPages()  // 获取总页数
Page.getTotalElements()	//获取总条数
```

