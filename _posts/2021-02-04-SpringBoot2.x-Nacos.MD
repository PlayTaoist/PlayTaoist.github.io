---
title:  "SpringBoot集成Nacos仅做动态配置中心-不重启服务"
categories: Nacos
tags: Nacos
author: LIUREN
---

# SpringBoot集成Nacos做动态配置中心-不重启服务

> Nacos 致力于帮助您发现、配置和管理微服务。Nacos 提供了一组简单易用的特性集，帮助您快速实现动态服务发现、服务配置、服务元数据及流量管理。他也可以作为动态配置中心来使用。下面我就当它是动态配置中心，做一个demo
>
> 协议：CC BY-SA 4.0 https://creativecommons.org/licenses/by-sa/4.0/  
>
> 版权声明：本文为原创文章，遵循 CC 4.0 BY-SA 版权协议，转载请附上原文出处链接及本声明。



## Nacos

如果此文章表述的不清晰可以参考官方的文档：https://nacos.io/zh-cn/docs/quick-start-spring-boot.html



## 下载Nacos的服务端版本

下载地址：nacos-server-1.4.1.zip 具体下载地址自行百度。github能下不过有点慢。

本应用主要在Windows 10上测试

然后解压

到nacos的bin目录下执行cmd命令行(此为测试，主要是单例启动)

```shell
startup.cmd -m standalone
```

![](https://www.codepeople.cn/imges/00x69.png)

新建一个Springboot的Maven项目如图

![](https://www.codepeople.cn/imges/00x70.png)

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
        <dependency>
		    <groupId>com.alibaba.boot</groupId>
		    <artifactId>nacos-config-spring-boot-starter</artifactId>
		    <version>${latest.version}</version>
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
                    <mainClass>com.hainan.www.SpringBootNacos.SpringBootNacosApplication</mainClass>
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

ConfigController.java

```java
package com.hainan.www.web;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.ResponseBody;
import org.springframework.web.bind.annotation.RestController;

import com.alibaba.nacos.api.config.annotation.NacosValue;

@RequestMapping("config")
@RestController
public class ConfigController {

	@NacosValue(value = "${useLocalCache:false}", autoRefreshed = true)
    private boolean useLocalCache;
	// 该方法是获取nacos服务端配置的dataCode编码，如果未配置该字段默认2021010，支持自动刷新
	@NacosValue(value = "${dataCode:2021010}", autoRefreshed = true)
	private String dataCode;
	
	@NacosValue(value = "${redisPort:6673}", autoRefreshed = true)
	private String redisPort;
	
	@NacosValue(value = "${spring.qq.name:6673}", autoRefreshed = true)
	private String springApplicationName;
	
	@NacosValue(value = "${spring.redis.host:6673}", autoRefreshed = true)
	private String springRedisHost;
	
    @GetMapping("/get")
    @ResponseBody
    public boolean get() {
    	System.out.println("useLocalCache==>" + useLocalCache);
    	
    	System.out.println("dataCode=>>>>>>" + dataCode);
    	
    	System.out.println("redisPort=>>>>>>" + redisPort);
    	
    	
    	System.out.println("springApplicationName=>>>>>>" + springApplicationName);
    	
    	System.out.println("springRedisHost=>>>>>>" + springRedisHost);
        return useLocalCache;
    }
}
```

application.yml

```yaml

spring:
  application:
    name: SpringBoot-Nacos
nacos:
  config:
    server-addr: 127.0.0.1:8848
```



首先确定nacos-server已经启动

然后启动Springboot项目

配置nacos-server端

登录地址如下：http://127.0.0.1:8848/           nacos/nacos



![](https://www.codepeople.cn/imges/00x72.png)

![](https://www.codepeople.cn/imges/00x71.png)

![](https://www.codepeople.cn/imges/00x73.png)



如此配置然后调用springboot的get方法

地址：

http://localhost:8080/config/get

看控制台打印信息如下：

```sh
useLocalCache==>true
dataCode=>>>>>>20210204
redisPort=>>>>>>6379
springApplicationName=>>>>>>SpringBoot-sss
springRedisHost=>>>>>>127.0.0.1:6379
```



主要注意的问题是：

nacos如果默认直接启动，使用的是其自带的derby数据库；如果想使用mysql数据库则需要到nacos-server-1.4.1.zip解压缩包下的conf下把下列注解解开

如果mysql是8.0版本的话，使用官网文档会报错需要在db.url.0=**********&allowPublicKeyRetrieval=true  加上`&allowPublicKeyRetrieval=true`

```properties
#*************** Config Module Related Configurations ***************#
### If use MySQL as datasource:
spring.datasource.platform=mysql

### Count of DB:
db.num=1

### Connect URL of DB:
db.url.0=jdbc:mysql://127.0.0.1:3306/nacos?characterEncoding=utf8&connectTimeout=1000&socketTimeout=3000&autoReconnect=true&useUnicode=true&useSSL=false&serverTimezone=UTC&allowPublicKeyRetrieval=true
db.user.0=root
db.password.0=root
```

同时把conf目录下的nacos-mysql.sql导入到mysql数据库中

这样就可以动态修改配置内容了，而且不用重启服务

Nacos对yaml格式解析不是特别好，注意使用yaml的配置格式容易获取不到对应的内容



博客地址：<https://www.codepeople.cn>

=====================================================================

微信公众号：

![](https://www.codepeople.cn/imges/weixin_icon/weixin.jpg)