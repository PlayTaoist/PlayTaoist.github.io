---
title:  "SpringCloud2.x的简单实践教程(一)"
categories: SpringCloud
tags: SpringCloud
author: LIUREN
---

# SpringCloud2.x的简单实践教程(一)

> 网上有很多springCloud1.x的教程，可以直接参考博客进行学习比如[**纯洁的微笑**](http://www.ityouknow.com/)的博客，讲解的很清晰。如果想要学springboot2.x的东西那么就要去网上直接找资料了，多看官方文档也是可以的
>



## 第一步　服务端配置。

使用`Spring Boot 2.0.2.RELEASE`和`Spring Cloud Finchley.SR2`官方正式版进行项目构建,创建名字为`eureka-server`的Spring Boot工程



`spring-cloud-starter-netflix-eureka-server`服务注册中心依赖

pom.xml

```
<project xmlns="http://maven.apache.org/POM/4.0.0"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<groupId>cn.codepeople.base</groupId>
	<artifactId>springcloud-base</artifactId>
	<version>1.0.0-SNAPSHOT</version>
	<packaging>pom</packaging>
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>2.0.2.RELEASE</version>
		<relativePath /> <!-- lookup parent from repository -->
	</parent>
	<repositories>
		<repository>
			<id>spring-snapshots</id>
			<name>Spring Snapshots</name>
			<url>https://repo.spring.io/libs-snapshot</url>
			<snapshots>
				<enabled>true</enabled>
			</snapshots>
		</repository>
		<repository>
			<id>spring-milestones</id>
			<name>Spring Milestones</name>
			<url>https://repo.spring.io/libs-milestone</url>
			<snapshots>
				<enabled>false</enabled>
			</snapshots>
		</repository>
		<repository>
			<id>spring-releases</id>
			<name>Spring Releases</name>
			<url>https://repo.spring.io/libs-release</url>
			<snapshots>
				<enabled>false</enabled>
			</snapshots>
		</repository>
	</repositories>

	<properties>
		<java.version>1.8</java.version>
	</properties>

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

	</dependencies>

	<dependencyManagement>
		<dependencies>
			<dependency>
				<groupId>org.springframework.cloud</groupId>
				<artifactId>spring-cloud-dependencies</artifactId>
				<version>Finchley.SR2</version>
				<type>pom</type>
				<scope>import</scope>
			</dependency>
		</dependencies>
	</dependencyManagement>

	<build>
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
			</plugin>
		</plugins>
	</build>
</project>
```





## 第二步  	创建注册中心

`springcloudserver` 

pom.xml

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<groupId>cn.codepeople</groupId>
	<artifactId>eureka-server</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<name>eureka-server</name>
	<description>Demo project for Spring Boot</description>

	<properties>
		<java.version>1.8</java.version>
	</properties>

	<parent>
		<groupId>cn.codepeople.base</groupId>
		<artifactId>springcloud-base</artifactId>
		<version>1.0.0-SNAPSHOT</version>
	</parent>
	<dependencies>
		<!-- eureka 服务端 -->
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
		</dependency>
	</dependencies>


</project>
```



## 第三步  	添加注册中心配置文件yml

application.yml

```yml
spring:
  application:
    name: eureka-server

server:
  port: 8761
```



## 第四步  	创建启动文件EurekaServerApplication.java

EurekaServerApplication.java

```java
package cn.codepeople;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.eureka.server.EnableEurekaServer;
/**
 * @description 注册服务中心
 * @author 刘仁
 * @2019年6月25日 14:52:13
 */
@EnableEurekaServer
@SpringBootApplication
public class EurekaServerApplication {

	public static void main(String[] args) throws Exception {
		SpringApplication.run(EurekaServerApplication.class, args);
	}

}
```



## 第五步   启动EurekaServerApplication.java注册中心

访问地址：`http://localhost:8761/`

![访问注册中心服务](https://www.codepeople.cn/imges/springcloud/001.png)

## 第六步   创建客户端

创建客户端`eureka-client`

pom.xml

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<groupId>cn.codepeople</groupId>
	<artifactId>eureka-client</artifactId>
	<version>0.0.1-SNAPSHOT</version>

	<parent>
		<groupId>cn.codepeople.base</groupId>
		<artifactId>springcloud-base</artifactId>
		<version>1.0.0-SNAPSHOT</version>
	</parent>

	<properties>
		<java.version>1.8</java.version>
	</properties>

	<dependencies>
		<!-- eureka 客户端 -->
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
		</dependency>
		<!-- springboot web模块 -->
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>
	</dependencies>

</project>
```



## 第七步   创建客户端的yml配置文件

application.yml

```yaml
spring:
  application:
    name: eureka-client
server:
  port: 8080

eureka:
  client:
    serviceUrl:
      defaultZone: http://localhost:8761/eureka/
```



## 第八步   编写Controller

HelloController.java

```java
package cn.codepeople.controller;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class HelloController {

	@GetMapping("/hello")
	public String index() {
		return "Hello World!";
	}
}
```



## 第九步    编写客户端启动程序

EurekaClientApplication.java

```java
package cn.codepeople;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
/**
 * @description 客户端
 * @author 刘仁
 * @2019年6月25日 14:52:13
 */
@SpringBootApplication
public class EurekaClientApplication {

	public static void main(String[] args) throws Exception {
		SpringApplication.run(EurekaClientApplication.class, args);
	}

}
```



然后刷新地址:`http://localhost:8761/`

效果：

![](https://www.codepeople.cn/imges/springcloud/002.png)





访问地址：`http://localhost:8080/hello`

![](https://www.codepeople.cn/imges/springcloud/003.png)

> 说明：以上就是全部的虚拟机CentOS 7.4 安装MySQL 8.0的安装步骤
>
> 参考文章：<https://blog.csdn.net/weixin_42266606/article/details/80879571>
>
> 参考文章：<https://www.cnblogs.com/phpandmysql/p/7912061.html>

