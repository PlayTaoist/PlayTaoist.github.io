---
title:  "springboot的快速集成多数据源的启动器"
categories: Springboot dynamic-datasource
tags: Springboot dynamic-datasource
author: LIUREN
---

# springboot的快速集成多数据源的启动器

> dynamic-datasource-spring-boot-starter 是一个基于springboot的快速集成多数据源的启动器。
> 其支持 **Jdk 1.7+, SpringBoot 1.4.x 1.5.x 2.0.x**。
> **示例项目** 可参考项目以下项目
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
		<version>2.2.6.RELEASE</version>
		<relativePath/> <!-- lookup parent from repository -->
	</parent>
	<groupId>cn.codepeople</groupId>
	<artifactId>dynamic-datasource-spring-boot-starter</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<name>dynamic-datasource-spring-boot-starter</name>
	<description>Demo project for Spring Boot</description>

	<properties>
		<java.version>1.8</java.version>
	</properties>

	<dependencies>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>

		<dependency>
			<groupId>mysql</groupId>
			<artifactId>mysql-connector-java</artifactId>
			<scope>runtime</scope>
		</dependency>
		<dependency>
			<groupId>org.projectlombok</groupId>
			<artifactId>lombok</artifactId>
			<optional>true</optional>
		</dependency>
		<!-- https://mvnrepository.com/artifact/com.baomidou/dynamic-datasource-spring-boot-starter -->
		<dependency>
		    <groupId>com.baomidou</groupId>
		    <artifactId>dynamic-datasource-spring-boot-starter</artifactId>
		    <version>3.0.0</version>
		</dependency>
		<dependency>
            <groupId>com.baomidou</groupId>
            <artifactId>mybatis-plus-boot-starter</artifactId>
            <version>2.2.0</version>
        </dependency>
        <dependency>
		    <groupId>com.github.xiaoymin</groupId>
		    <artifactId>knife4j-spring-boot-starter</artifactId>
		    <!--在引用时请在maven中央仓库搜索最新版本号-->
		    <version>2.0.2</version>
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

![](https://www.codepeople.cn/imges/00x15.png)

开始项目配置

application.yml

```properties
server:
  port: 8838
#数据源配置  
spring:
  datasource:
    dynamic:
      primary: master #设置默认的数据源或者数据源组,默认值即为master
      strict: false #设置严格模式,默认false不启动. 启动后在未匹配到指定数据源时候回抛出异常,不启动会使用默认数据源.
      datasource:
        master:
          url: jdbc:mysql://127.0.0.1:3306/master?useUnicode=true&zeroDateTimeBehavior=convertToNull&autoReconnect=true&useSSL=false&characterEncoding=utf-8&serverTimezone=UTC
          username: root
          password: root
          driver-class-name: com.mysql.cj.jdbc.Driver
        slave:
          url: jdbc:mysql://127.0.0.1:3306/salve?useUnicode=true&zeroDateTimeBehavior=convertToNull&autoReconnect=true&useSSL=false&characterEncoding=utf-8&serverTimezone=UTC
          username: root
          password: root
          driver-class-name: com.mysql.cj.jdbc.Driver
mybatis-plus:
  configuration:
    map-underscore-to-camel-case: true #开启驼峰命名
```

Springboot配置

DynamicDatasourceSpringBootStarterApplication.java

```java
package cn.codepeople.web;

import org.mybatis.spring.annotation.MapperScan;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
@MapperScan("cn.codepeople.web.mapper.**")
public class DynamicDatasourceSpringBootStarterApplication {

	public static void main(String[] args) {
		SpringApplication.run(DynamicDatasourceSpringBootStarterApplication.class, args);
	}

}
```

SwaggerConfiguration配置

SwaggerConfiguration.java

```java
package cn.codepeople.web.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Import;

import com.github.xiaoymin.knife4j.spring.annotations.EnableKnife4j;

import springfox.bean.validators.configuration.BeanValidatorPluginsConfiguration;
import springfox.documentation.builders.ApiInfoBuilder;
import springfox.documentation.builders.PathSelectors;
import springfox.documentation.builders.RequestHandlerSelectors;
import springfox.documentation.service.ApiInfo;
import springfox.documentation.spi.DocumentationType;
import springfox.documentation.spring.web.plugins.Docket;
import springfox.documentation.swagger2.annotations.EnableSwagger2;

@Configuration
@EnableSwagger2
@EnableKnife4j
@Import(BeanValidatorPluginsConfiguration.class)
public class SwaggerConfiguration {

	 @Bean(value = "defaultApi2")
	    public Docket defaultApi2() {
	        Docket docket=new Docket(DocumentationType.SWAGGER_2)
	                .apiInfo(apiInfo())
	                //分组名称
	                .groupName("2.X版本")
	                .select()
	                //这里指定Controller扫描包路径(项目路径也行)
	                .apis(RequestHandlerSelectors.basePackage("cn.codepeople.web"))
	                .paths(PathSelectors.any())
	                .build();
	        return docket;
	    }
	 
	    private ApiInfo apiInfo() {
	        return new ApiInfoBuilder()
	                .title("BaoMiDou动态数据源测试")
	                .description("BaoMiDou动态数据源测试接口")
	                .termsOfServiceUrl("http://localhost:8838/")
	                .version("1.0")
	                .build();
	    }
}
```

新建实体类

Log.java

```java
package cn.codepeople.web.entity;

import java.util.Date;

import io.swagger.annotations.ApiModel;
import io.swagger.annotations.ApiModelProperty;
import lombok.Data;
import lombok.NoArgsConstructor;

@Data
@NoArgsConstructor
@ApiModel(value = "日志模型")
public class Log {

	@ApiModelProperty(value = "id")
	private Long id;
	
	@ApiModelProperty(value = "logName")
	private String logName;
	
	@ApiModelProperty(value = "logDetail")
	private String logDetail;
	
	@ApiModelProperty(value = "createBy")
	private String createBy;
	
	@ApiModelProperty(value = "createTime")
	private Date createTime;
	
	@ApiModelProperty(value = "updateBy")
	private String updateBy;
	
	@ApiModelProperty(value = "updateTime")
	private Date updateTime;
	
	@ApiModelProperty(value = "remark")
	private String remark;
	
}
```

User.java

```java
package cn.codepeople.web.entity;

import java.util.Date;

import io.swagger.annotations.ApiModel;
import io.swagger.annotations.ApiModelProperty;
import lombok.Data;
import lombok.NoArgsConstructor;

@Data
@NoArgsConstructor
@ApiModel(value = "用户模型")
public class User {

	@ApiModelProperty(value = "id")
	private Long id;
	
	@ApiModelProperty(value = "userName")
	private String userName;
	
	@ApiModelProperty(value = "age")
	private Integer age;
	
	@ApiModelProperty(value = "createBy")
	private String createBy;
	
	@ApiModelProperty(value = "createTime")
	private Date createTime;
	
	@ApiModelProperty(value = "updateBy")
	private String updateBy;
	
	@ApiModelProperty(value = "updateTime")
	private Date updateTime;
	
	@ApiModelProperty(value = "remark")
	private String remark;
}
```

新建Service接口类

LogService.java

```java
package cn.codepeople.web.service;

import cn.codepeople.web.entity.Log;

public interface LogService {

	int insertLog(Log log);

}
```

UserService.java

```java
package cn.codepeople.web.service;

import cn.codepeople.web.entity.User;

public interface UserService {

	int insertUser(User user);

}
```





新建LogServiceImpl实现类

LogServiceImpl.java

```java
package cn.codepeople.web.service.impl;

import java.util.Date;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import com.baomidou.dynamic.datasource.annotation.DS;

import cn.codepeople.web.entity.Log;
import cn.codepeople.web.mapper.LogMapper;
import cn.codepeople.web.service.LogService;
@Service
public class LogServiceImpl implements LogService {

	@Autowired
	private LogMapper logMapper;
	
	@Override
	@DS("slave")
	public int insertLog(Log log) {
		log.setLogName("log_name");
		log.setLogDetail("log_detail");
		log.setCreateBy("adminOK");
		log.setCreateTime(new Date());
		log.setUpdateBy("adminOK");
		log.setUpdateTime(new Date());
		return logMapper.insertVo(log);
	}

}
```



新建UserServiceImpl实现类

UserServiceImpl.java

```java
package cn.codepeople.web.service.impl;

import java.util.Date;
import java.util.Random;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import cn.codepeople.web.entity.User;
import cn.codepeople.web.mapper.UserMapper;
import cn.codepeople.web.service.UserService;

@Service
public class UserServiceImpl implements UserService {

	@Autowired
	private UserMapper userMapper;
	
	@Override
	public int insertUser(User user) {
		Random random=new Random();
		user.setAge(random.nextInt(90)+10);
		user.setUserName("管理员123");
		user.setCreateBy("adminOK");
		user.setCreateTime(new Date());
		user.setUpdateBy("adminOK");
		user.setUpdateTime(new Date());
		return userMapper.insertVo(user);
	}

}
```



新建controller类

```java
package cn.codepeople.web.controller;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import cn.codepeople.web.entity.Log;
import cn.codepeople.web.entity.User;
import cn.codepeople.web.service.LogService;
import cn.codepeople.web.service.UserService;
import io.swagger.annotations.Api;
import io.swagger.annotations.ApiOperation;

@RestController
@RequestMapping("/user")
@Api(value = "动态数据源测试接口")
public class UserController {

	@Autowired
	private UserService userService;
	
	@Autowired
	private LogService logService;
	
	@ApiOperation(value = "添加用户", nickname = "添加用户接口", notes = "入参是复杂对象", produces = "application/json")
	@PostMapping("/user/insert")
	public int insert(@RequestBody User user) {
		return userService.insertUser(user);
	}
	
	@ApiOperation(value = "添加日志", nickname = "添加日志接口", notes = "入参是复杂对象", produces = "application/json")
	@PostMapping("/log/insert")
	public int insert(@RequestBody Log log) {
		return logService.insertLog(log);
	}
}
```



修改LogMapper.java类和xml文件

```java
package cn.codepeople.web.mapper;

import cn.codepeople.web.entity.Log;

public interface LogMapper {

	int insertVo(Log log);

}
```

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
"http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="cn.codepeople.web.mapper.LogMapper">
    
    <resultMap type="cn.codepeople.web.entity.Log" id="BaseResultMap">
        <result property="id"    column="id"    />
        <result property="logName"    column="log_name"    />
        <result property="logDetail"    column="log_detail"    />
        <result property="createBy"    column="create_by"    />
        <result property="createTime"    jdbcType="TIMESTAMP" column="create_time"    />
        <result property="updateBy"    column="update_by"    />
        <result property="updateTime"     jdbcType="TIMESTAMP" column="update_time"    />
        <result property="remark"    column="remark"    />
    </resultMap>

    <sql id="selectVo">
        select id, log_name, log_detail, create_by, create_time, update_by, update_time, remark from tb_log
    </sql>

    <select id="selectList" parameterType="cn.codepeople.web.entity.Log" resultMap="BaseResultMap">
        <include refid="selectVo"/>
        <where>  
            <if test="logName != null  and logName != ''"> and log_name like '%#{logName}%'</if>
        </where>
    </select>
    
    <select id="selectById" parameterType="String" resultMap="BaseResultMap">
        <include refid="selectVo"/>
        where id = #{id}
    </select>
        
    <insert id="insertVo" parameterType="cn.codepeople.web.entity.Log">
        insert into tb_log
        <trim prefix="(" suffix=")" suffixOverrides=",">
            <if test="id != null  and id != ''">id,</if>
            <if test="logName != null  and logName != ''">log_name,</if>
            <if test="logDetail != null  and logDetail != ''">log_detail,</if>
            <if test="createBy != null  and createBy != ''">create_by,</if>
            <if test="createTime != null ">create_time,</if>
            <if test="updateBy != null  and updateBy != ''">update_by,</if>
            <if test="updateTime != null ">update_time,</if>
            <if test="remark != null  and remark != ''">remark,</if>
         </trim>
        <trim prefix="values (" suffix=")" suffixOverrides=",">
            <if test="id != null  and id != ''">#{id},</if>
            <if test="logName != null  and logName != ''">#{logName},</if>
            <if test="logDetail != null  and logDetail != ''">#{logDetail},</if>
            <if test="createBy != null  and createBy != ''">#{createBy},</if>
            <if test="createTime != null ">#{createTime},</if>
            <if test="updateBy != null  and updateBy != ''">#{updateBy},</if>
            <if test="updateTime != null ">#{updateTime},</if>
            <if test="remark != null  and remark != ''">#{remark},</if>
         </trim>
    </insert>

    <update id="updateVo" parameterType="cn.codepeople.web.entity.Log">
        update tb_user
        <trim prefix="SET" suffixOverrides=",">
            <if test="logName != null  and logName != ''">log_name = #{logName},</if>
            <if test="logDetail != null  and logDetail != ''">log_detail = #{logDetail},</if>
            <if test="createBy != null  and createBy != ''">create_by = #{createBy},</if>
            <if test="createTime != null">create_time = #{createTime},</if>
            <if test="updateBy != null  and updateBy != ''">update_by = #{updateBy},</if>
            <if test="updateTime != null">update_time = #{updateTime},</if>
            <if test="remark != null and remark != ''">remark = #{remark},</if>
        </trim>
        where id = #{id}
    </update>

    <delete id="deleteById" parameterType="String">
        delete from tb_user where id = #{id}
    </delete>

    <delete id="deleteByIds" parameterType="String">
        delete from tb_user where id in 
        <foreach item="id" collection="array" open="(" separator="," close=")">
            #{id}
        </foreach>
    </delete>
    
</mapper>
```

修改UserMapper.java类和xml文件

```java
package cn.codepeople.web.mapper;

import cn.codepeople.web.entity.User;

public interface UserMapper {

	int insertVo(User user);

}
```

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
"http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="cn.codepeople.web.mapper.UserMapper">
    
    <resultMap type="cn.codepeople.web.entity.User" id="BaseResultMap">
        <result property="id"    column="id"    />
        <result property="userName"    column="user_name"    />
        <result property="age"    column="age"    />
        <result property="createBy"    column="create_by"    />
        <result property="createTime"   jdbcType="TIMESTAMP"  column="create_time"    />
        <result property="updateBy"    column="update_by"    />
        <result property="updateTime"    jdbcType="TIMESTAMP"  column="update_time"    />
        <result property="remark"    column="remark"    />
    </resultMap>

    <sql id="selectVo">
        select id, user_name, age, create_by, create_time, update_by, update_time, remark from tb_user
    </sql>

    <select id="selectList" parameterType="cn.codepeople.web.entity.User" resultMap="BaseResultMap">
        <include refid="selectVo"/>
        <where>  
            <if test="userName != null  and userName != ''"> and user_name like '%#{userName}%'</if>
        </where>
    </select>
    
    <select id="selectById" parameterType="String" resultMap="BaseResultMap">
        <include refid="selectVo"/>
        where id = #{id}
    </select>
        
    <insert id="insertVo" parameterType="cn.codepeople.web.entity.User">
        insert into tb_user
        <trim prefix="(" suffix=")" suffixOverrides=",">
            <if test="id != null  and id != ''">id,</if>
            <if test="userName != null  and userName != ''">user_name,</if>
            <if test="age != null  and age != ''">age,</if>
            <if test="createBy != null  and createBy != ''">create_by,</if>
            <if test="createTime != null ">create_time,</if>
            <if test="updateBy != null  and updateBy != ''">update_by,</if>
            <if test="updateTime != null ">update_time,</if>
            <if test="remark != null  and remark != ''">remark,</if>
         </trim>
        <trim prefix="values (" suffix=")" suffixOverrides=",">
            <if test="id != null  and id != ''">#{id},</if>
            <if test="userName != null  and userName != ''">#{userName},</if>
            <if test="age != null  and age != ''">#{age},</if>
            <if test="createBy != null  and createBy != ''">#{createBy},</if>
            <if test="createTime != null ">#{createTime},</if>
            <if test="updateBy != null  and updateBy != ''">#{updateBy},</if>
            <if test="updateTime != null ">#{updateTime},</if>
            <if test="remark != null  and remark != ''">#{remark},</if>
         </trim>
    </insert>

    <update id="updateVo" parameterType="cn.codepeople.web.entity.User">
        update tb_user
        <trim prefix="SET" suffixOverrides=",">
            <if test="userName != null  and userName != ''">user_name = #{userName},</if>
            <if test="age != null  and age != ''">age = #{age},</if>
            <if test="createBy != null  and createBy != ''">create_by = #{createBy},</if>
            <if test="createTime != null ">create_time = #{createTime},</if>
            <if test="updateBy != null  and updateBy != ''">update_by = #{updateBy},</if>
            <if test="updateTime != null ">update_time = #{updateTime},</if>
            <if test="remark != null and remark != ''">remark = #{remark},</if>
        </trim>
        where id = #{id}
    </update>

    <delete id="deleteById" parameterType="String">
        delete from tb_user where id = #{id}
    </delete>

    <delete id="deleteByIds" parameterType="String">
        delete from tb_user where id in 
        <foreach item="id" collection="array" open="(" separator="," close=")">
            #{id}
        </foreach>
    </delete>
    
</mapper>
```



测试接口<http://127.0.0.1:8838/doc.html>

根据地址进行访问：

添加用户接口

![](https://www.codepeople.cn/imges/00x16.png)

添加日志接口

![](https://www.codepeople.cn/imges/00x17.png)

对应的数据库的表接口分别是：tb_user.sql

```sql
CREATE TABLE `tb_user` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT '用户ID',
  `user_name` varchar(30) DEFAULT NULL COMMENT '用户昵称',
  `age` int(4) DEFAULT '0' COMMENT '用户年龄',
  `create_by` varchar(64) DEFAULT '' COMMENT '创建者',
  `create_time` datetime DEFAULT NULL COMMENT '创建时间',
  `update_by` varchar(64) DEFAULT '' COMMENT '更新者',
  `update_time` datetime DEFAULT NULL COMMENT '更新时间',
  `remark` varchar(500) DEFAULT NULL COMMENT '备注',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=2 DEFAULT CHARSET=utf8mb4 COMMENT='用户信息表';
```

tb_log.sql

```sql
CREATE TABLE `tb_log` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT '用户ID',
  `log_name` varchar(120) NOT NULL COMMENT '用户昵称',
  `log_detail` varchar(200) DEFAULT NULL COMMENT '用户年龄',
  `create_by` varchar(64) DEFAULT '' COMMENT '创建者',
  `create_time` datetime DEFAULT NULL COMMENT '创建时间',
  `update_by` varchar(64) DEFAULT '' COMMENT '更新者',
  `update_time` datetime DEFAULT NULL COMMENT '更新时间',
  `remark` varchar(500) DEFAULT NULL COMMENT '备注',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=2 DEFAULT CHARSET=utf8mb4 COMMENT='用户信息表';
```

主要想要切换到某个数据库就直接使用@DS("slave")



#  简介

dynamic-datasource-spring-boot-starter 是一个基于springboot的快速集成多数据源的启动器。

其支持 **Jdk 1.7+, SpringBoot 1.4.x 1.5.x 2.0.x**。

**示例项目** 可参考项目下的samples目录。

# 特性

1. 数据源分组，适用于多种场景 纯粹多库 读写分离 一主多从 混合模式。
2. 内置敏感参数加密和启动初始化表结构schema数据库database。
3. 提供对Druid，Mybatis-Plus，P6sy，Jndi的快速集成。
4. 简化Druid和HikariCp配置，提供全局参数配置。
5. 提供自定义数据源来源接口(默认使用yml或properties配置)。
6. 提供项目启动后增减数据源方案。
7. 提供Mybatis环境下的 **纯读写分离** 方案。
8. 使用spel动态参数解析数据源，如从session，header或参数中获取数据源。（多租户架构神器）
9. 提供多层数据源嵌套切换。（ServiceA >>> ServiceB >>> ServiceC，每个Service都是不同的数据源）
10. 提供 **不使用注解** 而 **使用 正则 或 spel** 来切换数据源方案（实验性功能）。
11. **基于seata的分布式事务支持。**

# 约定

1. 本框架只做 **切换数据源** 这件核心的事情，并**不限制你的具体操作**，切换了数据源可以做任何CRUD。
2. 配置文件所有以下划线 `_` 分割的数据源 **首部** 即为组的名称，相同组名称的数据源会放在一个组下。
3. 切换数据源可以是组名，也可以是具体数据源名称。组名则切换时采用负载均衡算法切换。
4. 默认的数据源名称为 **master** ，你可以通过 `spring.datasource.dynamic.primary` 修改。
5. 方法上的注解优先于类上注解。

#  使用方法

1.引入dynamic-datasource-spring-boot-starter。

```xml
<dependency>
  <groupId>com.baomidou</groupId>
  <artifactId>dynamic-datasource-spring-boot-starter</artifactId>
  <version>${version}</version>
</dependency>
```

2.配置数据源。

```properties
spring:
  datasource:
    dynamic:
      primary: master #设置默认的数据源或者数据源组,默认值即为master
      strict: false #设置严格模式,默认false不启动. 启动后在未匹配到指定数据源时候回抛出异常,不启动会使用默认数据源.
      datasource:
        master:
          url: jdbc:mysql://xx.xx.xx.xx:3306/dynamic
          username: root
          password: 123456
          driver-class-name: com.mysql.jdbc.Driver
        slave_1:
          url: jdbc:mysql://xx.xx.xx.xx:3307/dynamic
          username: root
          password: 123456
          driver-class-name: com.mysql.jdbc.Driver
        slave_2:
          url: ENC(xxxxx) # 内置加密,使用请查看详细文档
          username: ENC(xxxxx)
          password: ENC(xxxxx)
          driver-class-name: com.mysql.jdbc.Driver
          schema: db/schema.sql # 配置则生效,自动初始化表结构
          data: db/data.sql # 配置则生效,自动初始化数据
          continue-on-error: true # 默认true,初始化失败是否继续
          separator: ";" # sql默认分号分隔符
          
       #......省略
       #以上会配置一个默认库master，一个组slave下有两个子库slave_1,slave_2
```

```properties
# 多主多从                      纯粹多库（记得设置primary）                   混合配置
spring:                               spring:                               spring:
  datasource:                           datasource:                           datasource:
    dynamic:                              dynamic:                              dynamic:
      datasource:                           datasource:                           datasource:
        master_1:                             mysql:                                master:
        master_2:                             oracle:                               slave_1:
        slave_1:                              sqlserver:                            slave_2:
        slave_2:                              postgresql:                           oracle_1:
        slave_3:                              h2:                                   oracle_2:
```



3.使用 **@DS** 切换数据源。

**@DS** 可以注解在方法上和类上，**同时存在方法注解优先于类上注解**。

强烈建议只注解在service实现上。

| 注解          | 结果                                     |
| ------------- | ---------------------------------------- |
| 没有@DS       | 默认数据源                               |
| @DS("dsName") | dsName可以为组名也可以为具体某个库的名称 |

```
@Service
@DS("slave")
public class UserServiceImpl implements UserService {

  @Autowired
  private JdbcTemplate jdbcTemplate;

  public List<Map<String, Object>> selectAll() {
    return  jdbcTemplate.queryForList("select * from user");
  }
  
  @Override
  @DS("slave_1")
  public List<Map<String, Object>> selectByCondition() {
    return  jdbcTemplate.queryForList("select * from user where age >10");
  }
}
```



具体内容请参考：<https://gitee.com/baomidou/dynamic-datasource-spring-boot-starter>



博客地址：<https://www.codepeople.cn>

=====================================================================

微信公众号：

![](https://www.codepeople.cn/imges/weixin_icon/weixin.jpg)