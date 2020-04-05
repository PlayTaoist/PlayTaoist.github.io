---
title:  "SpringBoot2.x集成Kafka的实现"
categories: SpringBoot2.0 Kafka
tags: SpringBoot2.0 Kafka
author: LIUREN
---

# SpringBoot2.x集成Kafka的实现

> Kafka用来发送小消息信息给订阅者，消费者接收到对应的消息



## 安装环境

```shell
开发工具：STS
Maven版本：apache-maven-3.5.2
java jdk 1.8
MySQL版本：5.7
系统：CentOS7.4
```



## 一.独立安装Zookeeper

**注意:** 尽量不要使用Kafka自带的zookeeper，避免出现问题。首先去apche下载zookeeper

[下载地址：](https://archive.apache.org/dist/zookeeper/)<https://archive.apache.org/dist/zookeeper/>

下载版本：zookeeper-3.4.13.tar.gz

然后上传到CentOS 中

上传完毕后解压`tar zxf zookeeper-3.4.13.tar.gz`

修改文件夹名称`mv zookeeper-3.4.13 zookeeper`

在zookeeper的目录下创建data-logs目录 `mkdir data-logs`

然后进去配置文件修改信息`cd zookeeper/conf`

重命名配置文件 `mv zoo_sample.cfg zoo.cfg`

修改配置文件信息`dataDir=/home/ucenter/soft/zookeeper/data-logs`

保存并启动zookeeper `bin/zkServer.sh start`

放开端口2181 `firewall-cmd --zone=public --add-port=2181/tcp --permanent`

重新加载防火墙 `firewall-cmd --reload`

查看所有放开的端口`firewall-cmd --list-ports`



## 二.安装kafka

下载Kafka的压缩包

[下载地址：](http://archive.apache.org/dist/kafka/2.1.1/)<http://archive.apache.org/dist/kafka/2.1.1/>

下载的文件：kafka_2.11-2.1.1.tgz

然后上传到CentOS 中

上传完毕后解压`tar zxf kafka_2.11-2.1.1.tgz`

修改文件夹名称`mv kafka_2.11-2.1.1 kafka`

新建文件夹`mkdir kafka-logs`

进入配置文件目录`cd conf`

编辑配置文件 `vim server.properties`

修改配置文件`log.dirs=/home/ucenter/soft/kafka/kafka-logs`

如果zookeeper和kafka不在同一台电脑上还需要修改配置文件中zookeeper的信息

保存退出，准备启动Kafka应用

`./kafka-server-start.sh  -daemon ../config/server.properties &`



## 三.springboot2.0项目使用Kafka

首先引入包 pom.xml：

```xml
<!-- https://mvnrepository.com/artifact/org.springframework.kafka/spring-kafka -->
		<dependency>
		    <groupId>org.springframework.kafka</groupId>
		    <artifactId>spring-kafka</artifactId>
		</dependency>
```

然后修改配置文件

```yaml
spring:
### kafka
### producer 配置
  kafka:
    producer:
      bootstrap-servers: 127.0.0.1:9092
      key-serializer: org.apache.kafka.common.serialization.StringSerializer
      value-serializer: org.apache.kafka.common.serialization.StringSerializer
### consumer 配置 
    consumer:
      enable-auto-commit: true
      group-id: applog
      auto-offset-reset: latest
      bootstrap-servers: 127.0.0.1:9092
```

创建KafkaProducer.java

```java
package cn.codepeople.config;

import java.util.Date;
import java.util.UUID;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.kafka.core.KafkaTemplate;
import org.springframework.scheduling.annotation.EnableScheduling;
import org.springframework.scheduling.annotation.Scheduled;
import org.springframework.stereotype.Component;
import org.springframework.util.concurrent.ListenableFuture;

import com.alibaba.fastjson.JSONObject;

import cn.codepeople.dto.KafkaVO;


/**
 * kafka消息的生产者,消息的发送方
 * @author 刘仁
 *
 */
@Component
@EnableScheduling
public class KafkaProducer {

	@Autowired
    private KafkaTemplate<String, String> kafkaTemplate;

    /**
     * 定时任务
     */
    @Scheduled(cron = "00/1 * * * * ?")
    public void send(){
        String message = UUID.randomUUID().toString();
        KafkaVO vo = new KafkaVO();
        vo.setId(message);
        vo.setMsg("需要发送消息的内容");
        vo.setDate(new Date());
        ListenableFuture<?> future = kafkaTemplate.send("app_log",JSONObject.toJSONString(vo));
        String message2 = "第二种类的订阅消息发送";
        ListenableFuture<?> future2 = kafkaTemplate.send("app_log2",message2);
        future.addCallback(o -> System.out.println("send-消息发送成功：" + message), throwable -> System.out.println("消息发送失败：" + message));
        future2.addCallback(o -> System.out.println("send-消息发送成功：" + message2), throwable -> System.out.println("消息发送失败：" + message2));
    }
}
```

创建 KafkaConsumer

```java
package cn.codepeople.config;

import org.springframework.kafka.annotation.KafkaListener;
import org.springframework.stereotype.Component;
/**
 * kafka消息的消费者
 * @author 刘仁
 *
 */
@Component
public class KafkaConsumer {

	@KafkaListener(topics = {"app_log"})
    public void receive(String message){
        System.out.println("app_log--消费消息:" + message);
    }
	
	@KafkaListener(topics = {"app_log2"})
    public void receive2(String message){
        System.out.println("app_log2--消费消息:" + message);
    }
}
```



启动Application

```java
/**   
 * @author lr
 * @date 2019年3月4日 上午11:30:02 
 * @version V1.0.0   
 */
package cn.codepeople;

import org.mybatis.spring.annotation.MapperScan;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class SysApplication {

    public static void main(String[] args) {
        SpringApplication.run(SysApplication.class, args);
    }
}
```



查看控制台打印日志信息【日志级别控制为INFO】：因为DEBUG会不断的刷Kafka消息



启动后 定时任务 1秒钟发送一次消息

```verilog
send-消息发送成功：3676a2d0-9776-4ae9-b102-c0c90d67981c
send-消息发送成功：第二种类的订阅消息发送
app_log--消费消息:{"date":1553506807001,"id":"3676a2d0-9776-4ae9-b102-c0c90d67981c","msg":"需要发送消息的内容"}
app_log2--消费消息:第二种类的订阅消息发送
send-消息发送成功：9c417e14-75b6-47b9-a865-74d11aae1a58
send-消息发送成功：第二种类的订阅消息发送
app_log--消费消息:{"date":1553506808002,"id":"9c417e14-75b6-47b9-a865-74d11aae1a58","msg":"需要发送消息的内容"}
app_log2--消费消息:第二种类的订阅消息发送
```



==================================================================

[博客地址](https://www.codepeople.cn)：<https://www.codepeople.cn>

==================================================================

微信公众号：

![](https://www.codepeople.cn/imges/weixin_icon/weixin.jpg)