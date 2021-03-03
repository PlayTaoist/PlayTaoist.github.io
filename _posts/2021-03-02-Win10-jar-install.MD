---
title:  "Win10 系统下制作第三方jar方法"
categories: Win10 jar
tags: Win10 jar
author: LIUREN
---

# Win10 系统下制作第三方jar方法

> Win10 系统下制作第三方jar方法；打包称第三方jar放到本地仓库中也好，自己独立部署使用也好；怎样轻量化的做一个轻量功能的jar
>
> 协议：CC BY-SA 4.0 https://creativecommons.org/licenses/by-sa/4.0/  
>
> 版权声明：本文为原创文章，遵循 CC 4.0 BY-SA 版权协议，转载请附上原文出处链接及本声明。



## 1.开启Eclipse然后新建父项目

![](https://www.codepeople.cn/imges/00x83.png)

![](https://www.codepeople.cn/imges/00x84.png)

![](https://www.codepeople.cn/imges/00x85.png)

![](https://www.codepeople.cn/imges/00x86.png)

![](https://www.codepeople.cn/imges/00x87.png)

删除后，修改pom.xml文件，修改后如下

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.kelan</groupId>
    <artifactId>kelan</artifactId>
    <version>1.0.0</version>

    <name>kelan</name>
    <url>http://www.kelan.vip</url>
    <description>科澜管理系统</description>
    
    <properties>
        <kelan.version>1.0.0</kelan.version>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <java.version>1.8</java.version>
        <maven-jar-plugin.version>3.1.1</maven-jar-plugin.version>
    </properties>

    <!-- 依赖声明 -->
    <dependencyManagement>
        <dependencies>
        
            <!-- SpringBoot的依赖配置-->
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-dependencies</artifactId>
                <version>2.2.13.RELEASE</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>

        </dependencies>
    </dependencyManagement>

    <modules>
        <module>kelan-comment</module>
  </modules>
    <packaging>pom</packaging>


    <dependencies>

    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.1</version>
                <configuration>
                    <source>${java.version}</source>
                    <target>${java.version}</target>
                    <encoding>${project.build.sourceEncoding}</encoding>
                </configuration>
            </plugin>
        </plugins>
    </build>


</project>
```



![](https://www.codepeople.cn/imges/00x88.png)

![](https://www.codepeople.cn/imges/00x89.png)

![](https://www.codepeople.cn/imges/00x90.png)

![](https://www.codepeople.cn/imges/00x91.png)

![](https://www.codepeople.cn/imges/00x92.png)

制作module后修改pom.xml(主要做一些简单的清理,然后引入所需要的jar包)

```xml
<?xml version="1.0"?>
<project xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd" xmlns="http://maven.apache.org/POM/4.0.0"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
  <modelVersion>4.0.0</modelVersion>
  <parent>
    <groupId>com.kelan</groupId>
    <artifactId>kelan</artifactId>
    <version>1.0.0</version>
  </parent>
  <artifactId>kelan-comment</artifactId>
  <name>kelan-comment</name>
  <url>http://maven.apache.org</url>
  <properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
  </properties>
  
  <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-mongodb</artifactId>
        </dependency>
    </dependencies>
</project>
```

编写好逻辑代码后可以使用 Run As -> Maven install

如图

![](https://www.codepeople.cn/imges/00x93.png)

源码下载地址：https://files.cnblogs.com/files/lr393993507/xomment.zip



------

=====================================================================

微信公众号：

![](https://www.codepeople.cn/imges/weixin_icon/weixin.jpg)