---
layout:     post
title:      Storm的定时器整合mysql案例
date:       2018-1-23
author:     yunzhs
header-img: img/Fate of Princess.jpg
catalog: true
tags:
    - storm
typora-root-url: ..
typora-copy-images-to: ..\img\posts
---

## 一.功能需求

功能需求：实现每五秒钟打印出当前时间，并将发送出来的数据存入到mysql数据库当中

## 二.导入整合jar包

```
<dependency>
    <groupId>org.apache.storm</groupId>
    <artifactId>storm-core</artifactId>
    <version>1.1.1</version>
<scope>provided</scope>
</dependency>
<dependency>
			<groupId>org.apache.storm</groupId>
			<artifactId>storm-jdbc</artifactId>
			<version>1.1.1</version>
		</dependency>
		<!-- https://mvnrepository.com/artifact/mysql/mysql-connector-java -->
		<dependency>
		    <groupId>mysql</groupId>
		    <artifactId>mysql-connector-java</artifactId>
		    <version>5.1.38</version>
		</dependency>
<dependency>
		    <groupId>com.google.collections</groupId>
		    <artifactId>google-collections</artifactId>
		    <version>1.0</version>
		</dependency>

```

打包方式需要换成这个

```
	<plugin>
          <artifactId> maven-assembly-plugin </artifactId>
          <configuration>
               <descriptorRefs>
                    <descriptorRef>jar-with-dependencies</descriptorRef>
               </descriptorRefs>
               <archive>
                    <manifest>
                         <mainClass>cn.itcast.storm.demo2.JdbcTopo</mainClass>
                    </manifest>
               </archive>
          </configuration>
          <executions>
               <execution>
                    <id>make-assembly</id>
                    <phase>package</phase>
                    <goals>
                         <goal>single</goal>
                    </goals>
               </execution>
          </executions>
     </plugin>

```

## 三.创建mysql数据库log_monitor以及数据库表user

```
/*
SQLyog Ultimate v8.32 
MySQL - 5.6.22-log : Database - log_monitor
*********************************************************************
*/ 

/*!40101 SET NAMES utf8 */;

/*!40101 SET SQL_MODE=''*/;

/*!40014 SET @OLD_UNIQUE_CHECKS=@@UNIQUE_CHECKS, UNIQUE_CHECKS=0 */;
/*!40014 SET @OLD_FOREIGN_KEY_CHECKS=@@FOREIGN_KEY_CHECKS, FOREIGN_KEY_CHECKS=0 */;
/*!40101 SET @OLD_SQL_MODE=@@SQL_MODE, SQL_MODE='NO_AUTO_VALUE_ON_ZERO' */;
/*!40111 SET @OLD_SQL_NOTES=@@SQL_NOTES, SQL_NOTES=0 */;
CREATE DATABASE /*!32312 IF NOT EXISTS*/`log_monitor` /*!40100 DEFAULT CHARACTER SET utf8 */;

USE `log_monitor`;

/*Table structure for table `log_monitor_app` */


/*Table structure for table `user` */

DROP TABLE IF EXISTS `user`;

CREATE TABLE `user` (
  `userId` int(11) NOT NULL AUTO_INCREMENT,
  `name` varchar(1024) DEFAULT NULL,
  `age` int(5) DEFAULT NULL,
  PRIMARY KEY (`userId`)
) ENGINE=InnoDB AUTO_INCREMENT=10 DEFAULT CHARSET=utf8;

/*Data for the table `user` */

insert  into `user`(`userId`,`name`,`age`) values (1,'c',1),(2,'b',1),(3,'a',1),(4,'c',1),(5,'a',1),(6,'b',1),(7,'c',1),(8,'c',1),(9,'b',1);

/*!40101 SET SQL_MODE=@OLD_SQL_MODE */;
/*!40014 SET FOREIGN_KEY_CHECKS=@OLD_FOREIGN_KEY_CHECKS */;
/*!40014 SET UNIQUE_CHECKS=@OLD_UNIQUE_CHECKS */;
/*!40111 SET SQL_NOTES=@OLD_SQL_NOTES */;

```

## 四.创建随机数据发送的spout

```
public class RandomStrSpout extends BaseRichSpout{

	private SpoutOutputCollector collector;
	private String[] strArr ;
	private Random random ;
	
	
	@Override
	public void open(Map conf, TopologyContext context, SpoutOutputCollector collector) {
		this.collector = collector;
		strArr = new String[]{"a","b","c"};
		random = new Random();
	}

	@Override
	public void nextTuple() {
		try {
			String randomStr = strArr[random.nextInt(strArr.length)];
			collector.emit(new Values(randomStr));
			Thread.sleep(1000);
		} catch (InterruptedException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
	}

	@Override
	public void declareOutputFields(OutputFieldsDeclarer declarer) {
		declarer.declare(new Fields("randomStr"));
	}
}

```

## 五.定时任务bolt开发

```
public class TickTimeBolt extends BaseBasicBolt{
	
	private SimpleDateFormat format ;
	
	@Override
	public Map<String, Object> getComponentConfiguration() {
		Config config = new Config();
		config.put(config.TOPOLOGY_TICK_TUPLE_FREQ_SECS, 5);
		return config;
	}

	
	
	@Override
	public void prepare(Map stormConf, TopologyContext context) {
		super.prepare(stormConf, context);
		format = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
	}
	
	@Override
	public void execute(Tuple input, BasicOutputCollector collector) {
	//如何判断tuple是系统的tuple还是kafka发送过来的tuple
		if(input.getSourceComponent().equals(Constants.SYSTEM_COMPONENT_ID)
				&& input.getSourceStreamId().equals(Constants.SYSTEM_TICK_STREAM_ID)){
			System.out.println(format.format(new Date()));
		}else{
			String stringByField = input.getStringByField("randomStr");
			System.out.println("发送数据为"+stringByField);
			collector.emit(new Values(null,stringByField,1));
		}
	}

	/**
	 * 注意这里的字段，这里的字段名称一定要与mysq数据库当中的字段名称保持一致
	 */
	@Override
	public void declareOutputFields(OutputFieldsDeclarer declarer) {
// 这里发送出去的数据，字段，一定要与数据库当中的字段名保持一致
		declarer.declare(new Fields("userId","name","age"));
	}
}
```

## 六.主代码main方法开发

```
public class JdbcTopo {

	public static void main(String[] args) throws Exception {
		TopologyBuilder builder = new TopologyBuilder();
		builder.setSpout("randomStrSpout", new RandomStrSpout());
		builder.setBolt("tickTimeBolt", new TickTimeBolt()).localOrShuffleGrouping("randomStrSpout");
		
		Map hikariConfigMap = new HashMap();
		hikariConfigMap.put("dataSourceClassName","com.mysql.jdbc.jdbc2.optional.MysqlDataSource");
		hikariConfigMap.put("dataSource.url", "jdbc:mysql://172.16.43.67/log_monitor");
		hikariConfigMap.put("dataSource.user","root");
		hikariConfigMap.put("dataSource.password","admin");
		ConnectionProvider connectionProvider = new HikariCPConnectionProvider(hikariConfigMap);

		String tableName = "user";
		List<Column> columnSchema = Lists.newArrayList(  
				new Column("userId", java.sql.Types.INTEGER),
			    new Column("name", java.sql.Types.VARCHAR),  
			    new Column("age", java.sql.Types.INTEGER));  
		JdbcMapper simpleJdbcMapper = new SimpleJdbcMapper(columnSchema);
		/*JdbcInsertBolt userPersistanceBolt = new JdbcInsertBolt(connectionProvider, simpleJdbcMapper)
		                                    .withTableName("welc")
		                                    .withQueryTimeoutSecs(30);
		                                    Or*/
		JdbcInsertBolt userPersistanceBolt = new JdbcInsertBolt(connectionProvider, simpleJdbcMapper)
		                                    .withInsertQuery("insert into user values (?,?,?)")
		                                    .withQueryTimeoutSecs(30);                        
		builder.setBolt("jdbcBolt", userPersistanceBolt).fieldsGrouping("tickTimeBolt", new Fields("name","age"));
		Config conf = new Config();
		if(args !=null && args.length > 0){
			StormSubmitter submitter  = new StormSubmitter();
			submitter.submitTopology(args[0], conf, builder.createTopology());
		}else{
			StormTopology wc = builder.createTopology();
		        //1.本地模式提交
	        LocalCluster localCluster = new LocalCluster();
	        localCluster.submitTopology( "mywordcount", conf, wc);
		}
	}

}

```

