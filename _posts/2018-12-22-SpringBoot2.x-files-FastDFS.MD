---
title:  "SpringBoot2.0集成FastDFS"
categories: SpringBoot2.0
tags: SpringBoot2.0
author: LIUREN
---

# SpringBoot2.0集成FastDFS

> 前两篇整体上介绍了通过 Nginx 和 FastDFS 的整合来实现文件服务器。但是，在实际开发中对图片或文件的操作都是通过应用程序来完成的，因此，本篇将介绍 Spring Boot 整合 FastDFS 客户端来实现对图片/文件服务器的访问。
>
> 如果有不了解 FastDFS 的读者可以先浏览[《CentOS7 安装FastDFS分布式文件系统》](https://www.codepeople.cn/2019/01/14/CentOS7.4-FastDFS-install/)或是另行查阅网上相关资料。



## 一、整合编码

项目整体的代码结构图如下：

![](https://www.codepeople.cn/imges/FastDFS/016.png)

### 添加依赖

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>2.0.4.RELEASE</version>
		<relativePath /> <!-- lookup parent from repository -->
	</parent>
  <groupId>com.sql.tools</groupId>
  <artifactId>springboot-sql-tools</artifactId>
  <version>0.0.1-SNAPSHOT</version>
  
  <properties>
		<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
		<project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
		<java.version>1.8</java.version>
		<mybatis.spring.version>1.3.2</mybatis.spring.version>
		<com.alibaba.druid.version>1.1.10</com.alibaba.druid.version>
		<log4j.version>1.2.17</log4j.version>
		<pagehelper.version>1.2.5</pagehelper.version>
		<docker.image.prefix>kitty</docker.image.prefix>
		<fastjson.version>1.2.48</fastjson.version>
		<commons-lang3>3.4</commons-lang3>
		<oracle.version>11.2.0.4.0</oracle.version>
		<spring.boot.admin.version>2.0.0</spring.boot.admin.version>
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
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
		</dependency>
		<!-- spring aop -->
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-aop</artifactId>
		</dependency>
		<!-- mybatis -->
        <dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
            <version>${mybatis.spring.version}</version>
        </dependency>
        <!-- mysql -->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
        </dependency>
        <!-- https://mvnrepository.com/artifact/com.oracle/ojdbc6 -->
		<dependency>
		    <groupId>com.oracle</groupId>
		    <artifactId>ojdbc6</artifactId>
		    <version>${oracle.version}</version>
		</dependency>
		        
        <dependency>
		   <groupId>com.alibaba</groupId>
		   <artifactId>druid-spring-boot-starter</artifactId>
		   <version>${com.alibaba.druid.version}</version>
		</dependency>
		<!-- log4j -->
		<dependency>
		    <groupId>log4j</groupId>
		    <artifactId>log4j</artifactId>
		    <version>${log4j.version}</version>
		</dependency>
		<!-- swagger -->
		<dependency>
		    <groupId>io.springfox</groupId>
		    <artifactId>springfox-swagger2</artifactId>
		    <version>2.8.0</version>
		</dependency>
		<dependency>
		    <groupId>io.springfox</groupId>
		    <artifactId>springfox-swagger-ui</artifactId>
		    <version>2.8.0</version>
		</dependency>
		<!-- pagehelper -->
		<dependency>
		    <groupId>com.github.pagehelper</groupId>
		    <artifactId>pagehelper-spring-boot-starter</artifactId>
		    <version>${pagehelper.version}</version>
		</dependency>
		<!-- fastjson -->
		<dependency>
			<groupId>com.alibaba</groupId>
			<artifactId>fastjson</artifactId>
			<version>${fastjson.version}</version>
		</dependency>
		<dependency>
			<groupId>org.apache.commons</groupId>
			<artifactId>commons-lang3</artifactId>
		</dependency>
		<!-- https://mvnrepository.com/artifact/net.oschina.zcx7878/fastdfs-client-java -->
		<!-- fastdfs-client-java -->
		<dependency>
		    <groupId>com.github.tobato</groupId>
		    <artifactId>fastdfs-client</artifactId>
		    <version>1.26.5</version>
		    <exclusions>
		    	<exclusion>
		    		<groupId>ch.qos.logback</groupId>
		    		<artifactId>logback-classic</artifactId>
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
		<!-- 打包时拷贝MyBatis的映射文件 -->
        <resources>
            <resource>
                <directory>src/main/java</directory>
                <includes>
                    <include>**/sqlmap/*.xml</include>
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

**我在里面添加了Mybatis和数据连接驱动，使用的日志系统是`Solf4j`,`log4j2`**

配置文件如下：

application.yml

```properties
# Tomcat
server:
    tomcat:
        uri-encoding: UTF-8
        max-threads: 1000
        min-spare-threads: 30
    port: 8001
    session:
        timeout:7200
    context-path: /lion-admin
# DataSource
spring:
  datasource:
    name: druidDataSource
    type: com.alibaba.druid.pool.DruidDataSource
    druid:
      driver-class-name: oracle.jdbc.driver.OracleDriver
      url: jdbc:oracle:thin:@192.168.0.205:1521:orcl
      username: gcloud_check
      password: gcloud_check
      filters: stat,wall,log4j,config
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
```

端口:8001

数据使用的是：Oracle

**注意：** FastDFS的第三方客户端包引入的时候要去除logback的包，不然与log4j2冲突,启动会报错

```xml
<dependency>
		    <groupId>com.github.tobato</groupId>
		    <artifactId>fastdfs-client</artifactId>
		    <version>1.26.5</version>
		    <exclusions>
		    	<exclusion>
		    		<groupId>ch.qos.logback</groupId>
		    		<artifactId>logback-classic</artifactId>
		    	</exclusion>
		    </exclusions>
		</dependency>
```

上边的 fastdfs-client 是并非 FastDFS Client 原作者编写的整合包，具体详情可以访问 <https://github.com/tobato/FastDFS_Client>



log4j2-spring.xml

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
        <root level="INFO">
            <appender-ref ref="Console"/>
            <appender-ref ref="RollingFileInfo"/>
            <appender-ref ref="RollingFileWarn"/>
            <appender-ref ref="RollingFileError"/>
        </root>
    </loggers>
 
</configuration>
```

或者看这边配置log4j2的整合文章[SpringBoot 2.0 的Log4j2日志信息配置](https://www.codepeople.cn/2018/12/17/log4j2-logger-config/)

## 代码

```java
/**   
 * @author lr
 * @date 2018年12月26日 下午4:28:14 
 * @version V1.0.0   
 */
package com.louis.sql.tools;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.EnableMBeanExport;
import org.springframework.jmx.support.RegistrationPolicy;

@Configuration
@ComponentScan
@EnableMBeanExport(registration = RegistrationPolicy.IGNORE_EXISTING)
@SpringBootApplication
public class ToolsApplication {

    public static void main(String[] args) {
        SpringApplication.run(ToolsApplication.class, args);
    }
}
```

FastDFSClient.java

```java
/**   
 * @author lr
 * @date 2019年1月28日 下午1:41:39 
 * @version V1.0.0   
 */
package com.louis.sql.tools.config;

import java.io.ByteArrayInputStream;
import java.io.File;
import java.io.FileInputStream;
import java.io.IOException;
import java.nio.charset.Charset;

import org.apache.commons.io.FilenameUtils;
import org.apache.commons.lang3.StringUtils;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;
import org.springframework.web.multipart.MultipartFile;

import com.github.tobato.fastdfs.domain.conn.FdfsWebServer;
import com.github.tobato.fastdfs.domain.fdfs.StorePath;
import com.github.tobato.fastdfs.domain.proto.storage.DownloadByteArray;
import com.github.tobato.fastdfs.exception.FdfsUnsupportStorePathException;
import com.github.tobato.fastdfs.service.FastFileStorageClient;

@Component
public class FastDFSClient {

    private final Logger logger = LoggerFactory.getLogger(FastDFSClient.class);

    @Autowired
    private FastFileStorageClient storageClient;

    @Autowired
    private FdfsWebServer fdfsWebServer;

    /**
     * 上传文件
     * @param file 文件对象
     * @return 文件访问地址
     * @throws IOException
     */
    public String uploadFile(MultipartFile file) throws IOException {
        StorePath storePath = storageClient.uploadFile(file.getInputStream(),file.getSize(), FilenameUtils.getExtension(file.getOriginalFilename()),null);
        return getResAccessUrl(storePath);
    }

    /**
     * 上传文件
     * @param file 文件对象
     * @return 文件访问地址
     * @throws IOException
     */
    public String uploadFile(File file) throws IOException {
        FileInputStream inputStream = new FileInputStream (file);
        StorePath storePath = storageClient.uploadFile(inputStream,file.length(), FilenameUtils.getExtension(file.getName()),null);
        return getResAccessUrl(storePath);
    }

    /**
     * 将一段字符串生成一个文件上传
     * @param content 文件内容
     * @param fileExtension
     * @return
     */
    public String uploadFile(String content, String fileExtension) {
        byte[] buff = content.getBytes(Charset.forName("UTF-8"));
        ByteArrayInputStream stream = new ByteArrayInputStream(buff);
        StorePath storePath = storageClient.uploadFile(stream,buff.length, fileExtension,null);
        return getResAccessUrl(storePath);
    }

    // 封装图片完整URL地址
    private String getResAccessUrl(StorePath storePath) {
        String fileUrl = fdfsWebServer.getWebServerUrl() + storePath.getFullPath();
        return fileUrl;
    }
    
    /**
     * 下载文件
     * @param fileUrl 文件url
     * @return
     */
    public byte[]  download(String fileUrl) {
         String group = fileUrl.substring(0, fileUrl.indexOf("/"));
         String path = fileUrl.substring(fileUrl.indexOf("/") + 1);
         byte[] bytes = storageClient.downloadFile(group, path, new DownloadByteArray());
         return bytes;
    }

    /**
     * 删除文件
     * @param fileUrl 文件访问地址
     * @return
     */
    public void deleteFile(String fileUrl) {
        if (StringUtils.isEmpty(fileUrl)) {
            return;
        }
        try {
            StorePath storePath = StorePath.parseFromUrl(fileUrl);
            storageClient.deleteFile(storePath.getGroup(), storePath.getPath());
        } catch (FdfsUnsupportStorePathException e) {
            logger.warn(e.getMessage());
        }
    }

}
```

controller

```java
/**   
 * @author lr
 * @date 2019年1月28日 下午1:50:47 
 * @version V1.0.0   
 */
package com.louis.sql.tools.controller;

import java.net.URLEncoder;
import java.util.HashMap;
import java.util.Map;

import javax.servlet.ServletOutputStream;
import javax.servlet.http.HttpServletResponse;

import org.apache.commons.io.IOUtils;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.multipart.MultipartFile;

import com.louis.sql.tools.config.FastDFSClient;

@RestController
@RequestMapping("/fdfs")
public class FastDFSController {
        
        @Autowired
        private FastDFSClient fdfsClient;

        /**
         * 文件上传
         * @param file
         * @return
         * @throws Exception
         */
        @RequestMapping("/upload")
        public Map<String,Object> upload(MultipartFile file) throws Exception{
                
                String url = fdfsClient.uploadFile(file);
                
                Map<String,Object> result = new HashMap<>();
                result.put("code", 200);
                result.put("msg", "上传成功");
                result.put("url", url);
                
                return result;
        }
        
        /**
         * 文件下载
         * @param fileUrl  url 开头从组名开始
         * @param response
         * @throws Exception
         */
        @RequestMapping("/download")
        public void  download(String fileUrl, HttpServletResponse response) throws Exception{
                
                byte[] data = fdfsClient.download(fileUrl);
                
                response.setCharacterEncoding("UTF-8");
                response.setHeader("Content-disposition", "attachment;filename=" + URLEncoder.encode("test.jpg", "UTF-8"));
        
                // 写出
                ServletOutputStream outputStream = response.getOutputStream();
                IOUtils.write(data, outputStream);
        }
}
```

## 测试效果

然后通过swagger-ui的接口直接测试文件上传功能



访问地址：[swagger-ui首页]<http://localhost:8001/swagger-ui.html#>



![](https://www.codepeople.cn/imges/FastDFS/014.png)

![](https://www.codepeople.cn/imges/FastDFS/015.png)



如果需要看源码：<https://github.com/PlayTaoist/SpringBoot2-FastDFS>



=======================================================================================

微信公众号：

![](https://www.codepeople.cn/imges/weixin_icon/weixin.jpg)