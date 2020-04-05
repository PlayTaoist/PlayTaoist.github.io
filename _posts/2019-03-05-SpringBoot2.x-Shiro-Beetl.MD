---
title:  "SpringBoot2.0+Beetl+Shiro+Mybatis项目搭建"
categories: SpringBoot2.0 Shiro Beetl
tags: SpringBoot2.0 Shiro Beetl
author: LIUREN
---

# SpringBoot2.0+Beetl+Shiro+Mybatis项目搭建

> 学习SpringBoot2.0和Shiro的整合，顺便加入了Beetl



## 安装环境

```shell
开发工具：Eclipse
Maven版本：apache-maven-3.5.2
java jdk 1.8
MySQL版本：5.7
```



## 一.新建Maven项目

表的名称是：brand

![](https://www.codepeople.cn/imges/shiro/001.png)

![](https://www.codepeople.cn/imges/shiro/002.png)

## 二.在`pom.xml`中添加相应的架包

pom.xml内容如下：

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>2.0.4.RELEASE</version>
		<relativePath /> <!-- lookup parent from repository -->
	</parent>
	<groupId>cn.codepeople</groupId>
	<artifactId>springboot-shiro-beetl</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<properties>
		<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
		<project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
		<java.version>1.8</java.version>
		<log4j.version>1.2.17</log4j.version>
		<shiro.version>1.4.0</shiro.version>
		<beetl.version>1.1.73.RELEASE</beetl.version>
		<com.alibaba.druid.version>1.1.10</com.alibaba.druid.version>
		<mysql.version>5.1.47</mysql.version>
		<mybatis.version>2.0.0</mybatis.version>
	</properties>
	<dependencies>
		<!-- spring boot -->
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
			<exclusions>
				<exclusion>
					<groupId>org.springframework.boot</groupId>
					<artifactId>spring-boot-starter-logging</artifactId>
				</exclusion>
			</exclusions>
		</dependency>
		<!-- https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-log4j2 -->
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-log4j2</artifactId>
		</dependency>
		<!-- 引入shiro的spring整合框架 -->
		<dependency>
			<groupId>org.apache.shiro</groupId>
			<artifactId>shiro-spring</artifactId>
			<version>${shiro.version}</version>
		</dependency>
		<!-- https://mvnrepository.com/artifact/com.ibeetl/beetl-framework-starter -->
		<dependency>
			<groupId>com.ibeetl</groupId>
			<artifactId>beetl-framework-starter</artifactId>
			<version>${beetl.version}</version>
		</dependency>

		<dependency>
			<groupId>com.alibaba</groupId>
			<artifactId>druid-spring-boot-starter</artifactId>
			<version>${com.alibaba.druid.version}</version>
		</dependency>
		<!-- https://mvnrepository.com/artifact/mysql/mysql-connector-java -->
		<dependency>
			<groupId>mysql</groupId>
			<artifactId>mysql-connector-java</artifactId>
		</dependency>
		<!-- https://mvnrepository.com/artifact/org.mybatis.spring.boot/mybatis-spring-boot-starter -->
		<dependency>
			<groupId>org.mybatis.spring.boot</groupId>
			<artifactId>mybatis-spring-boot-starter</artifactId>
			<version>${mybatis.version}</version>
		</dependency>
		<!-- https://mvnrepository.com/artifact/org.projectlombok/lombok -->
		<dependency>
			<groupId>org.projectlombok</groupId>
			<artifactId>lombok</artifactId>
			<scope>provided</scope>
		</dependency>

	</dependencies>

	<build>
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
			</plugin>
		</plugins>
		<!-- 打包时拷贝MyBatis的映射文件 -->
		<resources>
			<resource>
				<directory>src/main/java</directory>
				<includes>
					<include>**/*.xml</include>
				</includes>
				<filtering>false</filtering>
			</resource>
			<resource>
				<directory>src/main/resources</directory>
				<includes>
					<include>**/*.*</include>
				</includes>
				<filtering>true</filtering>
			</resource>
		</resources>
	</build>
</project>
```

讲解：引入log4j2的包和配置文件

```xml
<!-- spring boot -->
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
			<exclusions>
				<exclusion>
					<groupId>org.springframework.boot</groupId>
					<artifactId>spring-boot-starter-logging</artifactId>
				</exclusion>
			</exclusions>
		</dependency>
		<!-- https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-log4j2 -->
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-log4j2</artifactId>
		</dependency>
```

引入beetl

```xml
<dependency>
			<groupId>com.ibeetl</groupId>
			<artifactId>beetl-framework-starter</artifactId>
			<version>${beetl.version}</version>
		</dependency>
```

引入druid的包

```xml
<dependency>
			<groupId>com.alibaba</groupId>
			<artifactId>druid-spring-boot-starter</artifactId>
			<version>${com.alibaba.druid.version}</version>
		</dependency>
```

引入mybatis包

```xml
<!-- https://mvnrepository.com/artifact/org.mybatis.spring.boot/mybatis-spring-boot-starter -->
		<dependency>
			<groupId>org.mybatis.spring.boot</groupId>
			<artifactId>mybatis-spring-boot-starter</artifactId>
			<version>${mybatis.version}</version>
		</dependency>
```

引入lombok包，可以省略GET和SET方法

```xml
<dependency>
			<groupId>org.projectlombok</groupId>
			<artifactId>lombok</artifactId>
			<scope>provided</scope>
		</dependency>
```

## 二.新增配置文件application.yml

application.yml

```yaml
# Tomcat
server:
    tomcat:
        uri-encoding: UTF-8
        max-threads: 1000
        min-spare-threads: 30
    port: 8002
    session:
        timeout:7200
    servlet:
      context-path: /sys
# DataSource
spring:
  datasource:
    name: druidDataSource
    type: com.alibaba.druid.pool.DruidDataSource
    druid:
      driver-class-name: com.mysql.jdbc.Driver
      url: jdbc:mysql://localhost:3306/springboot-shiro?useUnicode=true&zeroDateTimeBehavior=convertToNull&autoReconnect=true
      username: root
      password: root
      filters: stat,wall,slf4j,config
      max-active: 100
      initial-size: 1
      max-wait: 60000
      min-idle: 1
      time-between-eviction-runs-millis: 60000
      min-evictable-idle-time-millis: 300000
      validation-query: select 'x' FROM DUAL
      test-while-idle: true
      test-on-borrow: false
      test-on-return: false
      pool-prepared-statements: true
      max-open-prepared-statements: 50
      max-pool-prepared-statement-per-connection-size: 20
#mybatis的扫描包路径
mybatis:
  type-aliases-package: cn.codepeople.entity
  mapper-locations:  classpath:cn/codepeople/mapper/*.xml
# FastDFS
# ===================================================================
# 分布式文件系统FDFS配置
# ===================================================================
fdfs:
  so-timeout: 1501
  connect-timeout: 601 
  thumb-image:             #缩略图生成参数
    width: 150
    height: 150
  web-server-url: 192.168.0.137/
  tracker-list: 192.168.0.137:22122
#### beetl
beetl:
  suffix: html
```

**注意:** `filters: stat,wall,slf4j,config` 设置为slf4j的日志信息

banner.txt的内容

```tex
==== 欢迎使用SpringBoot2-Shiro的demo  ====
   _     _     _     _     _     _     _     _     _     _     _     _     _     _     _     _  
  / \   / \   / \   / \   / \   / \   / \   / \   / \   / \   / \   / \   / \   / \   / \   / \ 
 ( s ) ( p ) ( r ) ( i ) ( n ) ( g ) ( b ) ( o ) ( o ) ( t ) ( - ) ( s ) ( h ) ( i ) ( r ) ( o )
  \_/   \_/   \_/   \_/   \_/   \_/   \_/   \_/   \_/   \_/   \_/   \_/   \_/   \_/   \_/   \_/ 
                    
==== Lion Application == Version: 1.0   ====
==== 博客地址：https://www.codepeople.cn ====

```

log4j2-spring.xml配置文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!--设置log4j2的自身log级别为warn-->
<!--日志级别以及优先级排序: OFF > FATAL > ERROR > WARN > INFO > DEBUG > TRACE > ALL -->
<!--Configuration后面的status，这个用于设置log4j2自身内部的信息输出，可以不设置，
    当设置成trace时，你会看到log4j2内部各种详细输出-->
<!--monitorInterval：Log4j能够自动检测修改配置 文件和重新配置本身，设置间隔秒数-->
<configuration status="warn" monitorInterval="30">
    <!--先定义所有的appender-->
    <appenders>
        <!--这个输出控制台的配置-->
        <console name="Console" target="SYSTEM_OUT">
            <!--输出日志的格式-->
            <PatternLayout pattern="[%d{HH:mm:ss:SSS}] [%p] - %l - %m%n"/>
        </console>
        <!--文件会打印出所有信息，这个log每次运行程序会自动清空，由append属性决定，这个也挺有用的，适合临时测试用-->
        <File name="log" fileName="log/test.log" append="false">
            <PatternLayout pattern="%d{HH:mm:ss.SSS} %-5level %class{36} %L %M - %msg%xEx%n"/>
        </File>
        <!-- 这个会打印出所有的info及以下级别的信息，每次大小超过size，
        则这size大小的日志会自动存入按年份-月份建立的文件夹下面并进行压缩，作为存档-->
        <RollingFile name="RollingFileInfo" fileName="${sys:user.home}/logs/hpaasvc/info.log"
                     filePattern="${sys:user.home}/logs/hpaasvc/$${date:yyyy-MM}/info-%d{yyyy-MM-dd}-%i.log">
            <Filters>
                <!--控制台只输出level及以上级别的信息（onMatch），其他的直接拒绝（onMismatch）-->
                <ThresholdFilter level="INFO" onMatch="ACCEPT" onMismatch="DENY"/>
                <ThresholdFilter level="WARN" onMatch="DENY" onMismatch="NEUTRAL"/>
            </Filters>
            <PatternLayout pattern="[%d{HH:mm:ss:SSS}] [%p] - %l - %m%n"/>
            <Policies>
                <TimeBasedTriggeringPolicy/>
                <SizeBasedTriggeringPolicy size="100 MB"/>
            </Policies>
        </RollingFile>
 
        <RollingFile name="RollingFileWarn" fileName="${sys:user.home}/logs/hpaasvc/warn.log"
                     filePattern="${sys:user.home}/logs/hpaasvc/$${date:yyyy-MM}/warn-%d{yyyy-MM-dd}-%i.log">
            <Filters>
                <ThresholdFilter level="WARN" onMatch="ACCEPT" onMismatch="DENY"/>
                <ThresholdFilter level="ERROR" onMatch="DENY" onMismatch="NEUTRAL"/>
            </Filters>
            <PatternLayout pattern="[%d{HH:mm:ss:SSS}] [%p] - %l - %m%n"/>
            <Policies>
                <TimeBasedTriggeringPolicy/>
                <SizeBasedTriggeringPolicy size="100 MB"/>
            </Policies>
            <!-- DefaultRolloverStrategy属性如不设置，则默认为最多同一文件夹下7个文件，这里设置了20 -->
            <DefaultRolloverStrategy max="20"/>
        </RollingFile>
 
        <RollingFile name="RollingFileError" fileName="${sys:user.home}/logs/hpaasvc/error.log"
                     filePattern="${sys:user.home}/logs/hpaasvc/$${date:yyyy-MM}/error-%d{yyyy-MM-dd}-%i.log">
            <ThresholdFilter level="ERROR"/>
            <PatternLayout pattern="[%d{HH:mm:ss:SSS}] [%p] - %l - %m%n"/>
            <Policies>
                <TimeBasedTriggeringPolicy/>
                <SizeBasedTriggeringPolicy size="100 MB"/>
            </Policies>
        </RollingFile>
 
    </appenders>
    <!--然后定义logger，只有定义了logger并引入的appender，appender才会生效-->
    <loggers>
        <!--过滤掉spring和hibernate的一些无用的debug信息-->
        <logger name="org.springframework" level="INFO">
        </logger>
        <logger name="org.mybatis" level="INFO">
        </logger>
        <root level="DEBUG">
            <appender-ref ref="Console"/>
            <appender-ref ref="RollingFileInfo"/>
            <appender-ref ref="RollingFileWarn"/>
            <appender-ref ref="RollingFileError"/>
        </root>
    </loggers>
 
</configuration>
```

## 三.新增模板文件templates

项目结构文件路径

![](https://www.codepeople.cn/imges/shiro/003.png)

## 四.Beetl和Shiro整合

BeetlConf.java

```java
/**   
 * @author lr
 * @date 2019年3月3日 下午8:59:35 
 * @version V1.0.0   
 */
package cn.codepeople.shiro;

import org.beetl.core.GroupTemplate;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import com.ibeetl.starter.BeetlTemplateCustomize;

@Configuration
public class BeetlConf {

    @Bean
    public BeetlTemplateCustomize beetlTemplateCustomize() {
        return new BeetlTemplateCustomize() {
            public void customize(GroupTemplate groupTemplate) {
                //将实现了shiro标签的beetl方法注册到groupTemplate里
                groupTemplate.registerFunctionPackage("so", new ShiroExt());
            }
        };
    }

 }
```

ShiroExt.java

```java
/**   
 * @author lr
 * @date 2019年3月3日 下午8:58:12 
 * @version V1.0.0   
 */
package cn.codepeople.shiro;
import java.beans.BeanInfo;
import java.beans.Introspector;
import java.beans.PropertyDescriptor;
import java.util.Map;

import org.apache.shiro.SecurityUtils;
import org.apache.shiro.subject.Subject;
public class ShiroExt {
    /**
     * The guest tag
     * 
     * @return
     */
    public boolean isGuest() {
            return getSubject() == null || getSubject().getPrincipal() == null;
    }

    /**
     * The user tag
     * 
     * @return
     */
    public boolean isUser() {
            return getSubject() != null && getSubject().getPrincipal() != null;
    }

    /**
     * The authenticated tag
     * 
     * @return
     */
    public boolean isAuthenticated() {
            return getSubject() != null && getSubject().isAuthenticated();
    }

    public boolean isNotAuthenticated() {
            return !isAuthenticated();
    }

    /**
     * The principal tag
     * 
     * @param map
     * @return
     */
    public String principal(Map map) {
            String strValue = null;
            if (getSubject() != null) {

                    // Get the principal to print out
                    Object principal;
                    String type = map != null ? (String) map.get("type") : null;
                    if (type == null) {
                            principal = getSubject().getPrincipal();
                    } else {
                            principal = getPrincipalFromClassName(type);
                    }
                    String property = map != null ? (String) map.get("property") : null;
                    // Get the string value of the principal
                    if (principal != null) {
                            if (property == null) {
                                    strValue = principal.toString();
                            } else {
                                    strValue = getPrincipalProperty(principal, property);
                            }
                    }

            }

            if (strValue != null) {
                    return strValue;
            } else {
                    return null;
            }
    }

    /**
     * The hasRole tag
     * 
     * @param roleName
     * @return
     */
    public boolean hasRole(String roleName) {
            return getSubject() != null && getSubject().hasRole(roleName);
    }

    /**
     * The lacksRole tag
     * 
     * @param roleName
     * @return
     */
    public boolean lacksRole(String roleName) {
            boolean hasRole = getSubject() != null
                            && getSubject().hasRole(roleName);
            return !hasRole;
    }

    /**
     * The hasAnyRole tag
     * 
     * @param roleNames
     * @return
     */
    public boolean hasAnyRole(String roleNames) {
            boolean hasAnyRole = false;

            Subject subject = getSubject();

            if (subject != null) {

                    // Iterate through roles and check to see if the user has one of the
                    // roles
                    for (String role : roleNames.split(",")) {

                            if (subject.hasRole(role.trim())) {
                                    hasAnyRole = true;
                                    break;
                            }

                    }

            }

            return hasAnyRole;
    }

    /**
     * The hasPermission tag
     * 
     * @param p
     * @return
     */
    public boolean hasPermission(String p) {
            return getSubject() != null && getSubject().isPermitted(p);
    }

    /**
     * The lacksPermission tag
     * 
     * @param p
     * @return
     */
    public boolean lacksPermission(String p) {
            return !hasPermission(p);
    }

    @SuppressWarnings({ "unchecked" })
    private Object getPrincipalFromClassName(String type) {
            Object principal = null;

            try {
                    Class cls = Class.forName(type);
                    principal = getSubject().getPrincipals().oneByType(cls);
            } catch (ClassNotFoundException e) {

            }
            return principal;
    }

    private String getPrincipalProperty(Object principal, String property) {
            String strValue = null;

            try {
                    BeanInfo bi = Introspector.getBeanInfo(principal.getClass());

                    // Loop through the properties to get the string value of the
                    // specified property
                    boolean foundProperty = false;
                    for (PropertyDescriptor pd : bi.getPropertyDescriptors()) {
                            if (pd.getName().equals(property)) {
                                    Object value = pd.getReadMethod().invoke(principal,
                                                    (Object[]) null);
                                    strValue = String.valueOf(value);
                                    foundProperty = true;
                                    break;
                            }
                    }

                    if (!foundProperty) {
                            final String message = "Property [" + property
                                            + "] not found in principal of type ["
                                            + principal.getClass().getName() + "]";

                            throw new RuntimeException(message);
                    }

            } catch (Exception e) {
                    final String message = "Error reading property [" + property
                                    + "] from principal of type ["
                                    + principal.getClass().getName() + "]";

                    throw new RuntimeException(message, e);
            }

            return strValue;
    }

    protected Subject getSubject() {
            return SecurityUtils.getSubject();
    }


}
```



项目文件地址：<https://github.com/PlayTaoist/springboot-shiro-beetl>

=======================================================================================

博客地址：<https://www.codepeople.cn>

=======================================================================================

微信公众号：

![](https://www.codepeople.cn/imges/weixin_icon/weixin.jpg)