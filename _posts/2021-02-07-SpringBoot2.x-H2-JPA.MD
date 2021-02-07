---
title:  "SpringBoot集成H2和JPA"
categories: Nacos
tags: Nacos
author: LIUREN
---

# SpringBoot集成H2和JPA

> H2和derby作为一个内置数据库感觉非常好玩。上一篇讲了Springboot集成Nacos，当Nacos是单独运行未配置任何信息的时候，使用的数据库是derby；受到启发，如果我们自己做的一个小应用不想装其他什么的mysql这种大的。单单就是为了记录一下日志信息。我们可以考虑内置数据库H2.现在为大家做一篇整合过程，这个是在Springboot+Nacos基础上弄的。可称为Springboot + Nacos + H2 + JPA
>
> 协议：CC BY-SA 4.0 https://creativecommons.org/licenses/by-sa/4.0/  
>
> 版权声明：本文为原创文章，遵循 CC 4.0 BY-SA 版权协议，转载请附上原文出处链接及本声明。



## 内容说明

H2和derby作为一个内置数据库感觉非常好玩。上一篇讲了Springboot集成Nacos，当Nacos是单独运行未配置任何信息的时候，使用的数据库是derby；受到启发，如果我们自己做的一个小应用不想装其他什么的mysql这种大的。单单就是为了记录一下日志信息。我们可以考虑内置数据库H2.现在为大家做一篇整合过程，这个是在Springboot+Nacos基础上弄的。可称为Springboot + Nacos + H2 + JPA



## 引入对应的架包

pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.hainan.www</groupId>
    <artifactId>SpringBoot-Nacos</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>SpringBoot-Nacos</name>
    <description>Demo project for Spring Boot</description>

    <properties>
        <java.version>1.8</java.version>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <spring-boot.version>2.3.7.RELEASE</spring-boot.version>
        <latest.version>0.2.7</latest.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
        </dependency>
		<dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
            <scope>runtime</scope>
            <optional>true</optional>
        </dependency>
        
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-configuration-processor</artifactId>
            <optional>true</optional>
        </dependency>
        
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
        
        <!-- 阿里Nacos -->
        <dependency>
		    <groupId>com.alibaba.boot</groupId>
		    <artifactId>nacos-config-spring-boot-starter</artifactId>
		    <version>${latest.version}</version>
		</dependency>
		<dependency>
		        <groupId>org.springframework.boot</groupId>
		        <artifactId>spring-boot-starter-aop</artifactId>
		</dependency>
		
		<!-- H2内置数据库 -->
		<dependency>
		    <groupId>com.h2database</groupId>
		    <artifactId>h2</artifactId>
		    <scope>runtime</scope>
		</dependency>
		
		<!-- JPA -->
		<dependency>
		    <groupId>org.springframework.boot</groupId>
		    <artifactId>spring-boot-starter-data-jpa</artifactId>
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

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-dependencies</artifactId>
                <version>${spring-boot.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.8.1</version>
                <configuration>
                    <source>1.8</source>
                    <target>1.8</target>
                    <encoding>UTF-8</encoding>
                </configuration>
            </plugin>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <version>2.3.7.RELEASE</version>
                <configuration>
                    <mainClass>com.hainan.www.SpringBootNacosApplication</mainClass>
                </configuration>
                <executions>
                    <execution>
                        <id>repackage</id>
                        <goals>
                            <goal>repackage</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>

</project>
```

SpringBootNacosApplication.java

```java
package com.hainan.www;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

import com.alibaba.nacos.spring.context.annotation.config.NacosPropertySource;

@SpringBootApplication
@NacosPropertySource(dataId = "haiguan", autoRefreshed = true)
@NacosPropertySource(dataId = "redis", autoRefreshed = true)
public class SpringBootNacosApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringBootNacosApplication.class, args);
    }

}
```



application.yml

```yaml
nacos:
  config:
    server-addr: 127.0.0.1:8848
spring:
  application:
    name: SpringBoot-Nacos
  datasource:
    # data: classpath:db/data.sql  #进行该配置后，每次启动程序，程序都会运行resources/db/data.sql文件，对数据库的数据操作。
    driverClassName: org.h2.Driver  #配置JDBC Driver
    password: sa  #配置数据库密码
    platform: h2  #表明使用的数据库平台是h2
    schema: classpath:db/init.sql  #进行该配置后，每次启动程序，程序都会运行resources/db/schema.sql文件，对数据库的结构进行操作。'
    url: jdbc:h2:file:./db/sysopearlog #jdbc:h2:mem:dbtest  #配置h2数据库的连接地址
    username: sa  #配置数据库用户名
  h2:
    console:
      enabled: true  #进行该配置，程序开启时就会启动h2 web consloe。当然这是默认的，如果你不想在启动程序时启动h2 web consloe，那么就设置为false。'
      path: /h2  #进行该配置，你就可以通过YOUR_URL/h2访问h2 web consloe。YOUR_URL是你程序的访问URl。'
      settings:
        web-allow-others: true  # 进行该配置后，h2 web consloe就可以在远程访问了。否则只能在本机访问。'
  jpa:
    hibernate:
      ddl-auto: update  #设置ddl模式'
    show-sql: true #启用SQL语句的日志记录'
```



db目录下的sql脚本

```sql
create table if not exists sys_oper_log (
oper_id int not null primary key auto_increment,
title varchar(100),
business_type int(2),
method varchar(100),
request_method VARCHAR(100),
operator_type int(1),
oper_url VARCHAR(120),
oper_ip VARCHAR(100),
oper_location VARCHAR(100),
oper_param VARCHAR(1000),
json_result VARCHAR(1000),
status int(1),
error_msg VARCHAR(300),
oper_time DATE);
```



启动项目后，调用一下接口：然后查看数据库

![](https://www.codepeople.cn/imges/00x74.png)

![](https://www.codepeople.cn/imges/00x75.png)

博客地址：<https://www.codepeople.cn>

=====================================================================

微信公众号：

![](https://www.codepeople.cn/imges/weixin_icon/weixin.jpg)