---
title:  "SpringBoot集成ActiveMQ实例详解"
categories: ActiveMQ
tags: ActiveMQ
author: LIUREN
---

# SpringBoot集成ActiveMQ实例详解

> 在项目开发的过程中我们经常会遇到类似的业务场景：用户申请提现，后台进行账务处理、发送提现短信、调用银行打款通道。
>
> 协议：CC BY-SA 4.0 https://creativecommons.org/licenses/by-sa/4.0/  
>
> 版权声明：本文为原创文章，遵循 CC 4.0 BY-SA 版权协议，转载请附上原文出处链接及本声明。



## ActiveMQ

在这个过程中调用三方通道（短信或银行通道）都比较耗时，同时账务处理可能也是由专门的账务系统进行处理。那么，为了提高并发和相应速度，后面的三个操作都可以通过异步进行处理。这就用到了消息队列。

消息队列中间件是分布式系统中重要的组件，主要解决应用耦合、异步消息、流量削锋等问题，实现高性能、高可用、可伸缩和最终一致性架构，是大型分布式系统不可缺少的中间件。

市面上比较常见的消息队列有：ActiveMQ、RabbitMQ、ZeroMQ、Kafka、MetaMQ、RocketMQ。

在Spring Boot的starter中专门集成了ActiveMQ，因此，本篇文章我们就来讲讲对ActiveMQ的集成。



### JMS规范

JMS即Java消息服务（Java Message Service）应用程序接口，是一个Java平台中关于面向消息中间件（MOM）的API，用于在两个应用程序之间，或分布式系统中发送消息，进行异步通信。Java消息服务是一个与具体平台无关的API，绝大多数MOM提供商都对JMS提供支持。

JMS的消息机制有2种模型，一种是队列的形式（Point to Point—）发送的消息只能被一个消费者消费；一种是订阅（Topic）模式，可以被多个订阅者订阅，订阅者都会接收到同样的消息。

而ActiveMQ就是对JMS的实现之一。

### ActiveMQ介绍

ActiveMQ是一种开源的基于JMS（Java Message Servie）规范的一种消息中间件的实现，ActiveMQ的设计目标是提供标准的、面向消息的、能够跨越多语言和多系统的应用集成消息通信中间件。

它为企业应用中消息传递提供高可用、出色性能、可扩展、稳定和安全保障。

ActiveMQ实现JMS规范并在此之上提供大量额外的特性。ActiveMQ支持队列和订阅两种模式的消息发送。

AcitveMQ的数据传送流程如下图：

![img](https://www.codepeople.cn/imges/00x45.png)

ActiveMQ的两种消息传递类型：

（1）点对点传输，即一个生产者对应一个消费者，生产者向broke推送数据，数据存储在broke的一个队列中，当消费者接受该条队列里的数据。

（2）基于发布/订阅模式的传输，即根据订阅话题来接收相应数据，一个生产者可向多个消费者推送数据，与MQTT协议的实现是类似的。

两种消息传递类型的不同，点对点传输消费者可以接收到在连接之前生产者所推送的数据，而基于发布/订阅模式的传输方式消费者只能接收到连接之后生产者推送的数据。

### Spring Boot集成ActiveMQ

Spring Boot针对ActiveMQ专门提供了spring-boot-starter-activemq，用来支持ActiveMQ在Spring Boot的自动集成配置。在此基础上我们可以很轻易的进行集成和使用。

#### 创建项目并引入依赖

创建标准的Spring Boot项目，并在项目中引入以下依赖：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-activemq</artifactId>
</dependency>
```

此时如果不需要web或其他相关处理，只引入该依赖即可。如果使用pool的话, 就需要在pom中加入以下依赖：

```xml
<dependency>
     <groupId>org.apache.activemq</groupId>
     <artifactId>activemq-pool</artifactId>
</dependency>
```



#### 配置文件

在application.properties中添加如下配置：

```properties
# 基于内存的ActiveMQ
spring.activemq.in-memory=true
# 不使用连接池，如果使用连接池还需在pom中添加activemq-pool的依赖
spring.activemq.pool.enabled=false

# 独立安装的ActiveMQ
#spring.activemq.broker-url=tcp://127.0.0.1:61616
#spring.activemq.user=admin
#spring.activemq.password=admin
```

上述配置中有两套配置，Spring Boot支持基于内存ActiveMQ和基于独立安装的ActiveMQ。正常请求基于内存的形式是为了方便测试而使用，基于独立安装的形式才是真正用于生产环境。此处为了讲解功能，方便测试，采用基于内存的形式。

#### 队列模式实例

首先，我们来实现基于队列（Queue）形式的实现。这里需要用到两个类ActiveMQQueue和JmsMessagingTemplate。前者是由ActiveMQ对javax.jms.Queue的接口实现。后者为Spring提供发送消息的工具类，结合Queue对消息进行发送。

JmsMessagingTemplate默认已经被实例化，直接拿来使用即可。而ActiveMQQueue则需要我们进行实例化，并传入消息队列的名称。

```java
@Configuration
public class MyMqConfig {

  @Bean
  public Queue queue() {
    return new ActiveMQQueue("sms.queue");
  }
}
```

Spring Boot中很常规的实例化操作，不再赘述。当实例化完ActiveMQQueue之后，我们的队列便创建完成，下面创建对应的生产者和消费者。

生产者对应代码如下：

```java
@Component
public class Producer {

  @Resource
  private JmsMessagingTemplate jmsMessagingTemplate;

  @Resource
  private Queue queue;

  public void sendMsg(String msg) {
    System.out.println("发送消息内容 :" + msg);
    this.jmsMessagingTemplate.convertAndSend(this.queue, msg);
  }

}
```

此处用到JmsMessagingTemplate和Queue，上面已经提到，这两个类都已经完成了初始化。消费者对应的配置如下：

```java
@Component
public class Consumer {

  @JmsListener(destination = "sms.queue")
  public void receiveMsg(String text) {
    System.out.println("接收到消息 : "+text);
  }
}
```

Spring提供了注解式监听器端点：使用@JmsListener。使用@JmsListener托管bean的带注释方法对其进行订阅。在Java8中，@JmsListener是一个可重复的注解，可以关联多个JMS destinations到同一个方法中。而在Java 6和7中，可以使用@JmsListeners注解。

其中destination指定监控的消息队列名称为“sms.queue”。当队列sms.queue中有消息发送时会触发此方法的执行，text为消息内容。

上面完成了队列初始化、生产者和消费者代码的编写，下面通过单元测试来验证是否能够正确发送和处理消息。

```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class ActiveMqTests {

  @Autowired
  private Producer producer;

  @Test
  public void sendSimpleQueueMessage() {
    this.producer.sendMsg("提现200.00元");
  }
}
```

执行单元测试，会发现在日志中打印如下信息：

```tex
发送消息内容 :提现200.00元
接收到消息 : 提现200.00元
```

说明消息可以正常发送和接收。如果是基于内存模式，在执行单元测试时会打印出“javax.jms.JMSException: peer (vm://localhost#1) stopped.”异常日志，这是Info级别的错误，是ActiveMQ的一个bug。

#### 订阅模式实例

广播发送的消息，可以被多个消费者接收。这里我们就在原有的基础上进行广播消息的添加。

首先，Spring Boot集成ActiveMQ时默认只支持队列或者广播之一，通过配置项spring.jms.pub-sub-domain来指定，true 为广播模式，false为队列模式，默认情况下支持队列模式。

此时要使用广播模式，则需在配置文件中添加如下配置：

```properties
spring.jms.pub-sub-domain=true
```

需要注意的是，此时队列模式不可正常工作。

然后在MyMqConfig中添加：

```java
@Bean
public Topic topic() {
  return new ActiveMQTopic("sms.topic");
}
```

这里创建了ActiveMQTopic，并将topic的名称指定为sms.topic。

Producer中新增如下代码：

```java
@Resource
private Topic topic;

public void sendTopic(String msg) {
  System.out.println("发送Topic消息内容 :"+msg);
  this.jmsMessagingTemplate.convertAndSend(this.topic, msg);
}
```

为了演示多个广播接收者，在Comsumer中新增两个消费者：

```java
@JmsListener(destination = "sms.topic")
public void receiveTopic1(String text) {
  System.out.println("receiveTopic1接收到Topic消息 : " + text);
}

@JmsListener(destination = "sms.topic")
public void receiveTopic2(String text) {
  System.out.println("receiveTopic2接收到Topic消息 : " + text);
}
```

单元测试类中新增如下测试：

 ```java
@Test
public void sendSimpleTopicMessage() {
  this.producer.sendTopic("提现200.00元");
}
 ```

此时，执行单元测试，便可看到如下日志信息：

```reStructuredText
发送Topic消息内容 :提现200.00元
receiveTopic2接收到Topic消息 : 提现200.00元
receiveTopic1接收到Topic消息 : 提现200.00元
```

说明消息发送成功。

### 同时支持两种形式

在上面的实例中，要么支持队列模式要么支持广播模式，如果在生产环境中两者都需要支持，那么就需要自定义JmsListenerContainerFactory实例。当然，如果Spring Boot默认的配置无法满足需求，也可以自定义该类，这里只是其中场景之一。

基本配置和使用步骤：通过DefaultJmsListenerContainerFactory创建自定义的JmsListenerContainerFactory实例，在@JmsListener注解中通过containerFactory属性进行引用。

在MyMqConfig配置类中新增如下配置：

```java
@Bean("queueListenerFactory")
public JmsListenerContainerFactory<?> queueListenerFactory(ConnectionFactory connectionFactory) {
  DefaultJmsListenerContainerFactory factory = new DefaultJmsListenerContainerFactory();
  factory.setConnectionFactory(connectionFactory);
  factory.setPubSubDomain(false);
  return factory;
}

@Bean("topicListenerFactory")
public JmsListenerContainerFactory<?> topicListenerFactory(ConnectionFactory connectionFactory) {
  DefaultJmsListenerContainerFactory factory = new DefaultJmsListenerContainerFactory();
  factory.setConnectionFactory(connectionFactory);
  //设置为发布订阅方式, 默认情况下使用的生产消费者方式
  factory.setPubSubDomain(true);
  return factory;
}
```

这里分别实例化了基于队列和订阅的工厂类。然后分别在对应的消费者方法上添加containerFactory属性。示例代码如下：

```java
@JmsListener(destination = "sms.queue", containerFactory = "queueListenerFactory")
public void receiveMsg(String text) {
  System.out.println("接收到消息 : " + text);
}

@JmsListener(destination = "sms.topic", containerFactory = "topicListenerFactory")
public void receiveTopic1(String text) {
  System.out.println("receiveTopic1接收到Topic消息 : " + text);
}
```

分别执行两种形式的消息，发现都正常互利。同时，此时配置文件中的项spring.jms.pub-sub-domain也无效了。

### 其他事项

1、activeMq的端口号是61616；

2、使用topic,需要配置spring.jms.pub-sub-domain=true;

3、queue如果没有消费者，会将信息存储到queue中；

4、发送的消息为对象的时候，需要将对象序列化；消费者接收对象信息时需要使用ObjectMessage进行转化；

5、使用JmsListener注解中的containerFactory属性，可以配置spring.jms.pub-sub属性，实现同时接收queque和topic；

6、queue为点对点模式；tipic为发布订阅模式；

7、示例中的消息队列名称（sms.queue和sms.topic）可根据需要设置成配置属性；

源码地址：https://github.com/secbr/springboot-learn/tree/master/springboot-activemq

参考文章： 《[SpringBoot集成ActiveMQ实例详解](https://mp.weixin.qq.com/s?src=11&timestamp=1599636182&ver=2573&signature=otV5fKf4NanjGYY*4aNuvFVmmmKI-nrLZ-47RvuCJX6E*6trgsMCdXIt9Nn7cmqsVNYWOdyGcnCe8DbJYMzaLoTYroyEsc4ysEpMaR9UuAp*NgLIu-WiCUJ-WSjfU5dg&new=1)》
https://www.cnblogs.com/xiguadadage/p/11217604.html 
https://blog.csdn.net/bihansheng2010/article/details/87190645



博客地址：<https://www.codepeople.cn>

=====================================================================

微信公众号：

![](https://www.codepeople.cn/imges/weixin_icon/weixin.jpg)