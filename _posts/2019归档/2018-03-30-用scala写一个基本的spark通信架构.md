---
layout:     post
title:      使用Akka实现一个简易版的spark通信架构
date:       2017-09-27
author:     yunzhs
header-img: img/Fate of Princess.jpg
catalog: true
tags:
    - scala
typora-root-url: ..
typora-copy-images-to: ..\img\posts
---

## 1.架构图

![52239197214](/img/posts/1522391972145.png)

## 2.具体代码

**master代码**

```
import akka.actor.{Actor, ActorRef, ActorSystem, Props}
import com.typesafe.config.ConfigFactory

import scala.collection.mutable
import scala.collection.mutable.ListBuffer
import scala.concurrent.duration._


//todo:利用akka实现简易版的spark通信框架-----Master端

class Master  extends Actor{
  //构造代码块先被执行
  println("master constructor invoked")

  //定义一个map集合，用于存放worker信息
  private val workerMap = new mutable.HashMap[String,WorkerInfo]()
  //定义一个list集合，用于存放WorkerInfo信息，方便后期按照worker上的资源进行排序
  private val workerList = new ListBuffer[WorkerInfo]
  //master定时检查的时间间隔
  val CHECK_OUT_TIME_INTERVAL=15000 //15秒

  //prestart方法会在构造代码块执行后被调用，并且只被调用一次
  override def preStart(): Unit = {
    println("preStart method invoked")

      //master定时检查超时的worker
    //需要手动导入隐式转换
    import context.dispatcher
    context.system.scheduler.schedule(0 millis,CHECK_OUT_TIME_INTERVAL millis,self,CheckOutTime)
  }

  //receive方法会在prestart方法执行后被调用，表示不断的接受消息
  override def receive: Receive = {
    //master接受worker的注册信息
    case RegisterMessage(workerId,memory,cores) =>{
        //判断当前worker是否已经注册
      if(!workerMap.contains(workerId)){
        //保存信息到map集合中
        val workerInfo = new WorkerInfo(workerId,memory,cores)
        workerMap.put(workerId,workerInfo)
        //保存workerinfo到list集合中
        workerList +=workerInfo

        //master反馈注册成功给worker
        sender ! RegisteredMessage(s"workerId:$workerId 注册成功")
      }
    }
      //master接受worker的心跳信息
    case SendHeartBeat(workerId)=>{
      //判断worker是否已经注册，master只接受已经注册过的worker的心跳信息
      if(workerMap.contains(workerId)){
        //获取workerinfo信息
        val workerInfo: WorkerInfo = workerMap(workerId)
        //获取当前系统时间
        val lastTime: Long = System.currentTimeMillis()

        workerInfo.lastHeartBeatTime=lastTime
      }
    }
    case CheckOutTime=>{
      //过滤出超时的worker 判断逻辑： 获取当前系统时间 - worker上一次心跳时间 >master定时检查的时间间隔
        val outTimeWorkers: ListBuffer[WorkerInfo] = workerList.filter(x => System.currentTimeMillis() -x.lastHeartBeatTime > CHECK_OUT_TIME_INTERVAL)
      //遍历超时的worker信息，然后移除掉超时的worker
      for(workerInfo <- outTimeWorkers){
        //获取workerid
        val workerId: String = workerInfo.workerId
        //从map集合中移除掉超时的worker信息
        workerMap.remove(workerId)
        //从list集合中移除掉超时的workerInfo信息
        workerList -= workerInfo

        println("超时的workerId:" +workerId)
      }
      println("活着的worker总数：" + workerList.size)

      //master按照worker内存大小进行降序排列
     println(workerList.sortBy(x => x.memory).reverse.toList)
    }
  }
}
object Master{
  def main(args: Array[String]): Unit = {
    //master的ip地址
    val host=args(0)
    //master的port端口
    val port=args(1)

    //准备配置文件信息
    val configStr=
      s"""
         |akka.actor.provider = "akka.remote.RemoteActorRefProvider"
         |akka.remote.netty.tcp.hostname = "$host"
         |akka.remote.netty.tcp.port = "$port"
      """.stripMargin

    //配置config对象 利用ConfigFactory解析配置文件，获取配置信息
    val config=ConfigFactory.parseString(configStr)

    // 1、创建ActorSystem,它是整个进程中老大，它负责创建和监督actor，它是单例对象
    val masterActorSystem = ActorSystem("masterActorSystem",config)
    // 2、通过ActorSystem来创建master actor
    val masterActor: ActorRef = masterActorSystem.actorOf(Props(new Master),"masterActor")
    // 3、向master actor发送消息
    //masterActor ! "connect"
  }
}
```

**work代码**

```
import akka.actor.{Actor, ActorRef, ActorSystem, Props}
import com.typesafe.config.ConfigFactory

import scala.collection.mutable
import scala.collection.mutable.ListBuffer
import scala.concurrent.duration._


//todo:利用akka实现简易版的spark通信框架-----Master端

class Master  extends Actor{
  //构造代码块先被执行
  println("master constructor invoked")

  //定义一个map集合，用于存放worker信息
  private val workerMap = new mutable.HashMap[String,WorkerInfo]()
  //定义一个list集合，用于存放WorkerInfo信息，方便后期按照worker上的资源进行排序
  private val workerList = new ListBuffer[WorkerInfo]
  //master定时检查的时间间隔
  val CHECK_OUT_TIME_INTERVAL=15000 //15秒

  //prestart方法会在构造代码块执行后被调用，并且只被调用一次
  override def preStart(): Unit = {
    println("preStart method invoked")

      //master定时检查超时的worker
    //需要手动导入隐式转换
    import context.dispatcher
    context.system.scheduler.schedule(0 millis,CHECK_OUT_TIME_INTERVAL millis,self,CheckOutTime)
  }

  //receive方法会在prestart方法执行后被调用，表示不断的接受消息
  override def receive: Receive = {
    //master接受worker的注册信息
    case RegisterMessage(workerId,memory,cores) =>{
        //判断当前worker是否已经注册
      if(!workerMap.contains(workerId)){
        //保存信息到map集合中
        val workerInfo = new WorkerInfo(workerId,memory,cores)
        workerMap.put(workerId,workerInfo)
        //保存workerinfo到list集合中
        workerList +=workerInfo

        //master反馈注册成功给worker
        sender ! RegisteredMessage(s"workerId:$workerId 注册成功")
      }
    }
      //master接受worker的心跳信息
    case SendHeartBeat(workerId)=>{
      //判断worker是否已经注册，master只接受已经注册过的worker的心跳信息
      if(workerMap.contains(workerId)){
        //获取workerinfo信息
        val workerInfo: WorkerInfo = workerMap(workerId)
        //获取当前系统时间
        val lastTime: Long = System.currentTimeMillis()

        workerInfo.lastHeartBeatTime=lastTime
      }
    }
    case CheckOutTime=>{
      //过滤出超时的worker 判断逻辑： 获取当前系统时间 - worker上一次心跳时间 >master定时检查的时间间隔
        val outTimeWorkers: ListBuffer[WorkerInfo] = workerList.filter(x => System.currentTimeMillis() -x.lastHeartBeatTime > CHECK_OUT_TIME_INTERVAL)
      //遍历超时的worker信息，然后移除掉超时的worker
      for(workerInfo <- outTimeWorkers){
        //获取workerid
        val workerId: String = workerInfo.workerId
        //从map集合中移除掉超时的worker信息
        workerMap.remove(workerId)
        //从list集合中移除掉超时的workerInfo信息
        workerList -= workerInfo

        println("超时的workerId:" +workerId)
      }
      println("活着的worker总数：" + workerList.size)

      //master按照worker内存大小进行降序排列
     println(workerList.sortBy(x => x.memory).reverse.toList)
    }
  }
}
object Master{
  def main(args: Array[String]): Unit = {
    //master的ip地址
    val host=args(0)
    //master的port端口
    val port=args(1)

    //准备配置文件信息
    val configStr=
      s"""
         |akka.actor.provider = "akka.remote.RemoteActorRefProvider"
         |akka.remote.netty.tcp.hostname = "$host"
         |akka.remote.netty.tcp.port = "$port"
      """.stripMargin

    //配置config对象 利用ConfigFactory解析配置文件，获取配置信息
    val config=ConfigFactory.parseString(configStr)

    // 1、创建ActorSystem,它是整个进程中老大，它负责创建和监督actor，它是单例对象
    val masterActorSystem = ActorSystem("masterActorSystem",config)
    // 2、通过ActorSystem来创建master actor
    val masterActor: ActorRef = masterActorSystem.actorOf(Props(new Master),"masterActor")
    // 3、向master actor发送消息
    //masterActor ! "connect"
  }
}
```

workInfo类

```
package cn.itcast.spark

//封装worker信息
class WorkerInfo(val workerId:String,val memory:Int,val cores:Int) {
        //定义一个变量用于存放worker上一次心跳时间
      var lastHeartBeatTime:Long=_

  override def toString: String = {
    s"workerId:$workerId , memory:$memory , cores:$cores"
  }
}
```

样例类

```
package cn.itcast.spark

trait RemoteMessage  extends Serializable{

}

//worker向master发送注册信息，由于不在同一进程中，需要实现序列化
case class RegisterMessage(val workerId:String,val memory:Int,val cores:Int) extends RemoteMessage
//master反馈注册成功信息给worker,由于不在同一进程中，也需要实现序列化
case class RegisteredMessage(message:String) extends RemoteMessage
//worker向worker发送心跳 由于在同一进程中，不需要实现序列化
case object HeartBeat
//worker向master发送心跳，由于不在同一进程中，需要实现序列化
case class SendHeartBeat(val workerId:String) extends RemoteMessage
//master自己向自己发送消息，由于在同一进程中，不需要实现序列化
case object CheckOutTime

```

