---
title:  "SpringBoot2.x中使用@Scheduled创建定时任务"
categories: SpringBoot2.0
tags: SpringBoot2.0
author: LIUREN
---

# SpringBoot2.x中使用@Scheduled创建定时任务

> 在我们的项目开发过程中，经常需要定时任务来帮助我们来做一些内容，springboot默认已经帮我们实行了，只需要添加相应的注解就可以实现。



## POM包导入

pom包里面只需要引入springboot starter包即可

```xml
<dependencies>
	<dependency>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter</artifactId>
	</dependency>
	<dependency>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-test</artifactId>
		<scope>test</scope>
	</dependency>
     <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-devtools</artifactId>
        <optional>true</optional>
	</dependency>
</dependencies>
```

## 创建定时任务

在Spring Boot的主类中加入`@EnableScheduling`注解，启用定时任务的配置

```java

@SpringBootApplication
@EnableScheduling
public class Application {

	public static void main(String[] args) {
		SpringApplication.run(Application.class, args);
	}

}
```

## 创建定时任务实现类

定时任务1：

```java
@Component
public class SchedulerTask {

    private int count=0;

    @Scheduled(cron="*/6 * * * * ?")
    private void process(){
        System.out.println("this is scheduler task runing  "+(count++));
    }

}
```

定时任务2：

```java
@Component
public class Scheduler2Task {

    private static final SimpleDateFormat dateFormat = new SimpleDateFormat("HH:mm:ss");

    @Scheduled(fixedRate = 6000)
    public void reportCurrentTime() {
        System.out.println("现在时间：" + dateFormat.format(new Date()));
    }

}
```

结果如下：

```
this is scheduler task runing  0
现在时间：09:44:17
this is scheduler task runing  1
现在时间：09:44:23
this is scheduler task runing  2
现在时间：09:44:29
this is scheduler task runing  3
现在时间：09:44:35
```

## 参数说明

`@Scheduled` 参数可以接受两种定时的设置，一种是我们常用的`cron="*/6 * * * * ?"`,一种是 `fixedRate = 6000`，两种都表示每隔六秒打印一下内容。

**fixedRate 说明**

- `@Scheduled(fixedRate = 6000)` ：上一次开始执行时间点之后6秒再执行
- `@Scheduled(fixedDelay = 6000)` ：上一次执行完毕时间点之后6秒再执行
- `@Scheduled(initialDelay=1000, fixedRate=6000)` ：第一次延迟1秒后执行，之后按fixedRate的规则每6秒执行一次

参考文章来源：

文章来源一：<http://blog.didispace.com/springbootscheduled/>

文章来源二：<http://www.ityouknow.com/springboot/2016/12/02/spring-boot-scheduler.html>



# 定时任务详细讲解

## 定时任务概述

**定时任务：顾名思义就是在指定/特定的时间进行工作，比如我们的手机闹钟，它就是一种定时任务。**

> 实现方式

**Timer：** JDK自带的`java.util.Timer`；通过调度`java.util.TimerTask`的方式 **让程序按照某一个频度执行，但不能在指定时间运行。**一般用的较少。

**ScheduledExecutorService：** JDK1.5新增的，位于`java.util.concurrent`包中；是基于线程池设计的定时任务类，每个调度任务都会被分配到线程池中，并发执行，互不影响。

**Spring Task：** Spring3.0 以后新增了`task`，一个轻量级的`Quartz`，功能够用，用法简单。

**Quartz：** 功能最为强大的调度器，可以让程序在指定时间执行，也可以按照某一个频度执行，它还可以动态开关，但是配置起来比较复杂。现如今开源社区中已经很多`基于Quartz 实现的分布式定时任务项目`（[xxl-job](https://github.com/xuxueli/xxl-job/)、[elastic-job](https://github.com/elasticjob/elastic-job-lite)）。

## Timer 方式

基于 `Timer` 实现的定时调度，基本就是手撸代码，目前应用较少，不是很推荐

```java
package com.battcn.timer;

import java.time.LocalDateTime;
import java.util.Timer;
import java.util.TimerTask;

/**
 * 基于Timer实现的定时调度（不推荐，用该方式不如用 ScheduledExecutorService ）
 *
 * @author Levin
 * @since 2018/5/29 0029
 */
public class TimerDemo {

    public static void main(String[] args) {
        TimerTask timerTask = new TimerTask() {
            @Override
            public void run() {
                System.out.println("执行任务:" + LocalDateTime.now());
            }
        };
        Timer timer = new Timer();
        // timerTask：需要执行的任务
        // delay：延迟时间（以毫秒为单位）
        // period：间隔时间（以毫秒为单位）
        timer.schedule(timerTask, 5000, 3000);
    }
}
```

## 基于 ScheduledExecutorService

与`Timer`很类似，但它的效果更好，多线程并行处理定时任务时，`Timer`运行多个`TimeTask`时，**只要其中有一个因任务报错没有捕获抛出的异常，其它任务便会自动终止运行，使用 ScheduledExecutorService 则可以规避这个问题**

```java
package com.battcn.scheduled;

import java.time.LocalDateTime;
import java.util.concurrent.Executors;
import java.util.concurrent.ScheduledExecutorService;
import java.util.concurrent.TimeUnit;

/**
 * 基于 ScheduledExecutorService 方式,相对的比 Timer 要好
 *
 * @author Levin
 * @since 2018/5/29 0029
 */
public class ScheduledExecutorServiceDemo {

    public static void main(String[] args) {
        ScheduledExecutorService service = Executors.newScheduledThreadPool(10);
        // 参数：1、具体执行的任务   2、首次执行的延时时间
        //      3、任务执行间隔     4、间隔时间单位
        service.scheduleAtFixedRate(() -> System.out.println("执行任务A:" + LocalDateTime.now()), 0, 3, TimeUnit.SECONDS);
    }
}
```

## Spring Task(本章关键)

### 导入依赖

在 `pom.xml` 中添加 `spring-boot-starter-web` 依赖即可，它包含了`spring-context`，定时任务相关的就属于这个JAR下的`org.springframework.scheduling`包中

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
</dependencies>
```

### 定时任务

> `@Scheduled` 定时任务的核心

- **cron：** cron表达式，根据表达式循环执行，与`fixedRate`属性不同的是它是将时间进行了切割。（`@Scheduled(cron = "0/5 * * * * *")`任务将在`5、10、15、20...`这种情况下进行工作）
- **fixedRate：** 每隔多久执行一次；（`@Scheduled(fixedRate = 1000)` 假设第一次工作时间为`2018-05-29 16:58:28`，工作时长为`3秒`，那么下次任务的时候就是`2018-05-29 16:58:31`，配置成异步后，只要到了执行时间就会开辟新的线程工作），如果（`@Scheduled(fixedRate = 3000)` 假设第一次工作时间为`2018-05-29 16:58:28`，工作时长为`1秒`，那么下次任务的时间依然是`2018-05-29 16:58:31`）
- **fixedDelay：** 当前任务执行完毕后等待多久继续下次任务（`@Scheduled(fixedDelay = 3000)` 假设第一次任务工作时间为`2018-05-29 16:54:33`，工作时长为`5秒`，那么下次任务的时间就是`2018-05-29 16:54:41`）
- **initialDelay：** 第一次执行延迟时间，只是做延迟的设定，与`fixedDelay`关系密切，配合使用，相辅相成。

> `@Async` 代表该任务可以进行异步工作，由原本的串行改为并行

```java
package com.battcn.task;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.scheduling.annotation.Async;
import org.springframework.scheduling.annotation.Scheduled;
import org.springframework.stereotype.Component;

import java.time.LocalDateTime;

/**
 * 基于 Spring 自带的
 *
 * @author Levin
 * @since 2018/5/29 0029
 */
@Component
public class SpringTaskDemo {

    private static final Logger log = LoggerFactory.getLogger(SpringTaskDemo.class);

    @Async
    @Scheduled(cron = "0 0 0 1/1 * ?")
    public void scheduled1() throws InterruptedException {
        Thread.sleep(3000);
        log.info("scheduled1 每1天执行一次：{}", LocalDateTime.now());
    }

    @Scheduled(fixedRate = 1000)
    public void scheduled2() throws InterruptedException {
        Thread.sleep(3000);
        log.info("scheduled2 每1秒执行一次：{}", LocalDateTime.now());
    }

    @Scheduled(fixedDelay = 3000)
    public void scheduled3() throws InterruptedException {
        Thread.sleep(5000);
        log.info("scheduled3 上次执行完毕后隔3秒继续执行：{}", LocalDateTime.now());
    }

}
```

**cron表达式在线生成：** <http://www.bejson.com/othertools/cron/>

### 主函数

`@EnableScheduling` 注解表示开启对`@Scheduled`注解的解析；同时`new ThreadPoolTaskScheduler()`也是相当的关键，通过阅读过源码可以发现默认情况下的 `private volatile int poolSize = 1;`这就导致了多个任务的情况下容易出现竞争情况（多个任务的情况下，如果第一个任务没执行完毕，后续的任务将会进入等待状态）。

`@EnableAsync` 注解表示开启`@Async`注解的解析；作用就是将串行化的任务给并行化了。（`@Scheduled(cron = "0/1 * * * * *")`假设第一次工作时间为`2018-05-29 17:30:55`，工作周期为`3秒`；如果不加`@Async`那么下一次工作时间就是`2018-05-29 17:30:59`；如果加了`@Async` 下一次工作时间就是`2018-05-29 17:30:56`）

```java
package com.battcn;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.annotation.Bean;
import org.springframework.scheduling.TaskScheduler;
import org.springframework.scheduling.annotation.EnableAsync;
import org.springframework.scheduling.annotation.EnableScheduling;
import org.springframework.scheduling.concurrent.ThreadPoolTaskScheduler;


/**
 * @author Levin
 */
@EnableAsync
@EnableScheduling
@SpringBootApplication
public class Chapter15Application {

    public static void main(String[] args) {
        SpringApplication.run(Chapter15Application.class, args);
    }

    /**
     * 很关键：默认情况下 TaskScheduler 的 poolSize = 1
     *
     * @return 线程池
     */
    @Bean
    public TaskScheduler taskScheduler() {
        ThreadPoolTaskScheduler taskScheduler = new ThreadPoolTaskScheduler();
        taskScheduler.setPoolSize(10);
        return taskScheduler;
    }
}
```

### 测试

完成准备事项后，启动`Chapter15Application`，观察日志信息如下

```verilog
2018-05-29 17:35:51.479  INFO 32640 --- [taskScheduler-1] com.battcn.task.SpringTaskDemo           : scheduled2 每1秒执行一次：2018-05-29T17:35:51.479
2018-05-29 17:35:52.005  INFO 32640 --- [taskScheduler-3] com.battcn.task.SpringTaskDemo           : scheduled1 每1秒执行一次：2018-05-29T17:35:52.005
2018-05-29 17:35:53.002  INFO 32640 --- [taskScheduler-5] com.battcn.task.SpringTaskDemo           : scheduled1 每1秒执行一次：2018-05-29T17:35:53.002
2018-05-29 17:35:53.468  INFO 32640 --- [taskScheduler-2] com.battcn.task.SpringTaskDemo           : scheduled3 上次执行完毕后隔3秒继续执行：2018-05-29T17:35:53.468
2018-05-29 17:35:54.002  INFO 32640 --- [taskScheduler-6] com.battcn.task.SpringTaskDemo           : scheduled1 每1秒执行一次：2018-05-29T17:35:54.002
2018-05-29 17:35:54.479  INFO 32640 --- [taskScheduler-7] com.battcn.task.SpringTaskDemo           : scheduled2 每1秒执行一次：2018-05-29T17:35:54.479
2018-05-29 17:35:55.002  INFO 32640 --- [taskScheduler-8] com.battcn.task.SpringTaskDemo           : scheduled1 每1秒执行一次：2018-05-29T17:35:55.002
2018-05-29 17:35:56.001  INFO 32640 --- [taskScheduler-1] com.battcn.task.SpringTaskDemo           : scheduled1 每1秒执行一次：2018-05-29T17:35:56.001
2018-05-29 17:35:57.001  INFO 32640 --- [taskScheduler-3] com.battcn.task.SpringTaskDemo           : scheduled1 每1秒执行一次：2018-05-29T17:35:57.001
2018-05-29 17:35:57.479  INFO 32640 --- [taskScheduler-7] com.battcn.task.SpringTaskDemo           : scheduled2 每1秒执行一次：2018-05-29T17:35:57.479
2018-05-29 17:35:58.003  INFO 32640 --- [taskScheduler-4] com.battcn.task.SpringTaskDemo           : scheduled1 每1秒执行一次：2018-05-29T17:35:58.003
2018-05-29 17:35:59.001  INFO 32640 --- [taskScheduler-5] com.battcn.task.SpringTaskDemo           : scheduled1 每1秒执行一次：2018-05-29T17:35:59.001
2018-05-29 17:36:00.002  INFO 32640 --- [taskScheduler-6] com.battcn.task.SpringTaskDemo           : scheduled1 每1秒执行一次：2018-05-29T17:36:00.002
2018-05-29 17:36:00.480  INFO 32640 --- [taskScheduler-7] com.battcn.task.SpringTaskDemo           : scheduled2 每1秒执行一次：2018-05-29T17:36:00.480
2018-05-29 17:36:01.001  INFO 32640 --- [taskScheduler-8] com.battcn.task.SpringTaskDemo           : scheduled1 每1秒执行一次：2018-05-29T17:36:01.001
2018-05-29 17:36:01.470  INFO 32640 --- [taskScheduler-9] com.battcn.task.SpringTaskDemo           : scheduled3 上次执行完毕后隔3秒继续执行：2018-05-29T17:36:01.470
```

