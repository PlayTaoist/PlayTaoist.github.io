---
title:  "Springboot根据url后缀返回json或者xml或者html"
categories: Springboot json xml
tags: Springboot json xml
author: LIUREN
---

# Springboot根据url后缀返回json或者xml或者html

> Springboot根据url后缀.
> 返回json或者xml；
> 根据后缀名称返回html。
> 作者：liuren
>
> 版权声明：本文为原创文章，遵循 CC 4.0 BY-SA 版权协议，转载请附上原文出处链接及本声明。



## 初始化Springboot项目

## 一、首先引入的pom.xml文件包

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>2.2.4.RELEASE</version>
		<relativePath/> <!-- lookup parent from repository -->
	</parent>
	<groupId>cn.codepeople</groupId>
	<artifactId>springboot-json-xml</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<name>springboot-json-xml</name>
	<description>Springboot根据url后缀返回json或者xml或者html</description>

	<properties>
		<java.version>1.8</java.version>
	</properties>

	<dependencies>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-thymeleaf</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>

		<dependency>
            <groupId>com.fasterxml.jackson.dataformat</groupId>
            <artifactId>jackson-dataformat-xml</artifactId>
        </dependency>
        
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-devtools</artifactId>
			<scope>runtime</scope>
			<optional>true</optional>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
			<exclusions>
				<exclusion>
					<groupId>org.junit.vintage</groupId>
					<artifactId>junit-vintage-engine</artifactId>
				</exclusion>
			</exclusions>
		</dependency>
	</dependencies>

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

项目结构图

![](https://www.codepeople.cn/imges/00x11.png)

开始项目配置

application.properties

```properties
spring.thymeleaf.mode=HTML  #后缀名称
spring.thymeleaf.encoding=utf-8  #编码格式
spring.thymeleaf.cache=false #是否使用缓存
```

Springboot配置

WebMvcConfig.java

```java
package cn.codepeople.web.config;

import org.springframework.context.annotation.Configuration;
import org.springframework.http.MediaType;
import org.springframework.web.servlet.config.annotation.ContentNegotiationConfigurer;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurationSupport;

@Configuration
public class WebMvcConfig extends WebMvcConfigurationSupport {

	@Override
	protected void configureContentNegotiation(ContentNegotiationConfigurer configurer) {
		configurer.favorPathExtension(true) //是否支持后缀的方式
			.parameterName("mediaType")
		   .defaultContentType(MediaType.APPLICATION_JSON) 
		   .mediaType("xml", MediaType.APPLICATION_XML)   //当后缀名称为xml的时候返回xml数据
		   .mediaType("html", MediaType.TEXT_HTML)              //当后缀名称是html时候返回html页面
		   .mediaType("json", MediaType.APPLICATION_JSON);//当后缀名称是json的时候返回json数据
	}

}
```

新建Bean实体类类

User.java

```java
package cn.codepeople.web.entity;

import java.util.Date;

import javax.xml.bind.annotation.XmlRootElement;

@XmlRootElement
public class User {

	private Long id;
	
	private String userName;
	
	private String age;
	
	private Date  birthdy;
	
	private boolean delFlag;

	public Long getId() {
		return id;
	}

	public void setId(Long id) {
		this.id = id;
	}

	public String getUserName() {
		return userName;
	}

	public void setUserName(String userName) {
		this.userName = userName;
	}

	public String getAge() {
		return age;
	}

	public void setAge(String age) {
		this.age = age;
	}

	public Date getBirthdy() {
		return birthdy;
	}

	public void setBirthdy(Date birthdy) {
		this.birthdy = birthdy;
	}

	public boolean isDelFlag() {
		return delFlag;
	}

	public void setDelFlag(boolean delFlag) {
		this.delFlag = delFlag;
	}
	
	
	
}
```

新建UserService.java接口

UserService.java

```java
package cn.codepeople.web.service;

import java.util.List;

import cn.codepeople.web.entity.User;

public interface UserService {

	List<User> listUser();

}
```

新建UserServiceImpl实现类

UserServiceImpl.java

```java
package cn.codepeople.web.service.impl;

import java.util.ArrayList;
import java.util.Date;
import java.util.List;

import org.springframework.stereotype.Service;

import cn.codepeople.web.entity.User;
import cn.codepeople.web.service.UserService;

@Service
public class UserServiceImpl implements UserService {

	@Override
	public List<User> listUser() {
		List<User> list = new ArrayList<User>();
		User user = new User();
		user.setId(1L);
		user.setAge("12");
		user.setDelFlag(true);
		user.setUserName("zhang1");
		user.setBirthdy(new Date());
		list.add(user);
		User userx = new User();
		userx.setId(2L);
		userx.setAge("23");
		userx.setDelFlag(true);
		userx.setUserName("zhang2");
		userx.setBirthdy(new Date());
		list.add(userx);
		return list;
	}

}
```



新建controller类

```java
package cn.codepeople.web.controller;

import java.util.List;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.ResponseBody;

import cn.codepeople.web.entity.User;
import cn.codepeople.web.service.UserService;

@Controller
@RequestMapping("/user")
public class UserController {

	@Autowired
	private UserService userService;
	
	@RequestMapping("/list")
	@ResponseBody
	private List<User> listUser(){
		
		return userService.listUser();
		
	}
	
	@RequestMapping("/index")
	public String user() {
		return "list.html";
	}
}
```



修改SpringbootJsonXmlApplication.java启动类

```java
package cn.codepeople.web;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class SpringbootJsonXmlApplication {

	public static void main(String[] args) {
		SpringApplication.run(SpringbootJsonXmlApplication.class, args);
	}

}

```

templates目录下新建list.html文件

list.html

```html
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>Insert title here</title>
</head>
<body>
Springboot根据url后缀返回json或者xml或者html ->>>
</body>
</html>
```



启动成功页面

访问地址是：<http://127.0.0.1:8080/user/list.json>

![](https://www.codepeople.cn/imges/00x12.png)

访问地址是：<http://127.0.0.1:8080/user/list.xml

![](https://www.codepeople.cn/imges/00x13.png)

访问地址是：<http://127.0.0.1:8080/user/index.html

![](https://www.codepeople.cn/imges/00x14.png)

有一点我没明白，如果大家有深刻的理解欢迎帮我解答一下。WebMvcConfig.java中默认格式是json

但如果不加后缀返回的是xml格式文件，是因为我没在controller中做默认格式配置吗？如果不加后缀按照

WebMvcConfig.java配置不是应该返回json嘛？

博客地址：<https://www.codepeople.cn>

=====================================================================

微信公众号：

![](https://www.codepeople.cn/imges/weixin_icon/weixin.jpg)