---
title:  "SpringBoot2.0整合Mybatis"
categories: SpringBoot2.0
tags: SpringBoot2.0
author: LIUREN
---

# SpringBoot2.0整合Mybatis

> 本文记录了如何使用SpringBoot2.0整合Mybatis



## ORM对比图

以下针对**Spring JDBC、Spring Data Jpa、Mybatis**三款框架做了个粗略的对比。**一般应用的性能瓶颈并不是在于ORM，所以这三个框架技术选型应该考虑项目的场景、团队的技能掌握情况、开发周期(开发效率)…**

| 框架对比 | Spring JDBC | Spring Data Jpa | Mybatis |
| -------- | ----------- | --------------- | ------- |
| 性能     | 性能最好    | 性能最差        | 居中    |
| 代码量   | 多          | 少              | 多      |
| 学习成本 | 低          | 高              | 居中    |
| 推荐指数 | ❤❤❤         | ❤❤❤❤❤           | ❤❤❤❤❤   |

> **个人观点**

抛开学习成本而言，对于业务简单的中小型项目中使用`Spring Data Jpa` 开发无异于是最快速的。但是鉴于国内市场环境而言，掌握`Mybatis`无异于是佳的选择，低学习成本和动态SQL解耦的特点使得更容易被人们所接受。对于业务复杂且对性能要求较高的项目来说`Mybatis`往往能更好的胜任，可以自己进行SQL优化，同时更让我喜欢的是[Mybatis分页插件](https://gitee.com/free/Mybatis_PageHelper)与[通用Mapper(单表CURD无需自己手写)](https://gitee.com/free/Mapper)有了这两款插件的支持，还有什么理由拒绝`Mybatis`呢

## 导入依赖

在 `pom.xml` 中添加 `Mybatis` 的依赖包`mybatis-spring-boot-starter`，该包拥有自动装配的特点

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
	<groupId>com.louis.springboot.demo</groupId>
	<artifactId>springboot-demo</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<packaging>jar</packaging>

	<properties>
		<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
		<project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
		<java.version>1.8</java.version>
		<mybatis.spring.version>1.3.2</mybatis.spring.version>
		<com.alibaba.druid.version>1.1.10</com.alibaba.druid.version>
		<log4j.version>1.2.17</log4j.version>
		<pagehelper.version>1.2.5</pagehelper.version>
		<shiro.version>1.4.0</shiro.version>
		<kaptcha.version>0.0.9</kaptcha.version>
		<docker.image.prefix>kitty</docker.image.prefix>
		<fastjson.version>1.2.48</fastjson.version>
		<commons-lang3>3.4</commons-lang3>
		<spring.boot.admin.version>2.0.0</spring.boot.admin.version>
	</properties>

	<dependencies>
		<!-- spring boot -->
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
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
		<!--spring整合elasticsearch包-->
        <dependency>  
           <groupId>org.springframework.boot</groupId>  
           <artifactId>spring-boot-starter-data-elasticsearch</artifactId>  
       </dependency>
		<!--spring-boot-admin -->
		<dependency>
			<groupId>de.codecentric</groupId>
			<artifactId>spring-boot-admin-starter-client</artifactId>
			<version>${spring.boot.admin.version}</version>
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
		<!-- shiro -->
		<dependency>
		    <groupId>org.apache.shiro</groupId>
		     <artifactId>shiro-spring</artifactId>
		    <version>${shiro.version}</version>
		</dependency>
		<!-- 验证码 -->
        <dependency>
		    <groupId>com.github.axet</groupId>
		    <artifactId>kaptcha</artifactId>
		    <version>${kaptcha.version}</version>
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
			<version>${commons-lang3}</version>
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

## 连接数据库

与**SpringDataJpa、Spring JDBC**一样，需要在`application.properties`中添加数据源的配置，同时也需要添加对`mybatis`的配置

在 `application.properties` 文件中分别添加上**数据库、Mybatis、通用Mapper、PageHelper**的属性配置，这里只提供了常见场景的配置，更全的配置可以参考上文所述的**文文档(#^.^#)**

```properties
spring.datasource.url=jdbc:mysql://localhost:3306/chapter7?useUnicode=true&characterEncoding=UTF-8&zeroDateTimeBehavior=convertToNull&allowMultiQueries=true&useSSL=false
spring.datasource.password=root
spring.datasource.username=root
# 如果想看到mybatis日志需要做如下配置
logging.level.com.battcn=DEBUG
########## Mybatis 自身配置 ##########
mybatis.mapper-locations=classpath:com/battcn/mapper/*.xml
mybatis.type-aliases-package=com.battcn.entity
# 驼峰命名规范 如：数据库字段是  order_id 那么 实体字段就要写成 orderId
mybatis.configuration.map-underscore-to-camel-case=true
########## 通用Mapper ##########
# 主键自增回写方法,默认值MYSQL,详细说明请看文档
mapper.identity=MYSQL
mapper.mappers=tk.mybatis.mapper.common.BaseMapper
# 设置 insert 和 update 中，是否判断字符串类型!=''
mapper.not-empty=true
# 枚举按简单类型处理
mapper.enum-as-simple-type=true
########## 分页插件 ##########
pagehelper.helper-dialect=mysql
pagehelper.params=count=countSql
pagehelper.reasonable=false
pagehelper.support-methods-arguments=true
```



```yaml
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
      driver-class-name: com.mysql.jdbc.Driver
      url: jdbc:mysql://localhost:3306/kitty?useUnicode=true&zeroDateTimeBehavior=convertToNull&autoReconnect=true
      username: root
      password: root
      filters: stat,wall,log4j,config
      max-active: 100
      initial-size: 1
      max-wait: 60000
      min-idle: 1
      time-between-eviction-runs-millis: 60000
      min-evictable-idle-time-millis: 300000
      validation-query: select 'x'
      test-while-idle: true
      test-on-borrow: false
      test-on-return: false
      pool-prepared-statements: true
      max-open-prepared-statements: 50
      max-pool-prepared-statement-per-connection-size: 20
  data:
    elasticsearch:
      cluster-name: elasticsearch
      cluster-nodes: 127.0.0.1:9300
      repositories:
        enabled: true
# pagehelper   
pagehelper:
    helperDialect: mysql
    reasonable: true
    supportMethodsArguments: true
    params: count=countSql
```

> 通用Mapper

- **mapper.enum-as-simple-type：** 枚举按简单类型处理，如果有枚举字段则需要加上该配置才会做映射
- **mapper.not-empty：** 设置以后，会去判断 insert 和 update 中符串类型!=’’

> 分页插件

- **pagehelper.reasonable：** 分页合理化参数，默认值为false。当该参数设置为 true 时，pageNum<=0 时会查询第一页， pageNum>pages（超过总数时），会查询最后一页。默认false 时，直接根据参数进行查询。
- **support-methods-arguments：** 支持通过 Mapper 接口参数来传递分页参数，默认值false，分页插件会从查询方法的参数值中，自动根据上面 params 配置的字段中取值，查找到合适的值时就会自动分页。

> 注意事项

由于 **mybatis.mapper-locations=classpath:com/battcn/mapper/\*.xml**配置的在`java package`中，而`Spring Boot`默认只打入`java package -> *.java`，所以我们需要给`pom.xml`文件添加如下内容

```xml
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
```

## 具体编码

完成基本配置后，接下来进行具体的编码操作。

### 实体类

`通用Mapper`采用了JPA规范包中的注解，这种的设计避免了重复造轮子，更是让`Spring Data Jpa`的应用可以轻松切换到`Mybatis`

```java
package com.louis.kitty.admin.model;

import java.util.Date;

/**
 * 基础模型
 * @author Louis
 * @date Sep 13, 2018
 */
public class BaseModel {

	private Long id;
	
    private String createBy;

    private Date createTime;

    private String lastUpdateBy;

    private Date lastUpdateTime;

	public Long getId() {
		return id;
	}

	public void setId(Long id) {
		this.id = id;
	}

	public String getCreateBy() {
		return createBy;
	}

	public void setCreateBy(String createBy) {
		this.createBy = createBy;
	}

	public Date getCreateTime() {
		return createTime;
	}

	public void setCreateTime(Date createTime) {
		this.createTime = createTime;
	}

	public String getLastUpdateBy() {
		return lastUpdateBy;
	}

	public void setLastUpdateBy(String lastUpdateBy) {
		this.lastUpdateBy = lastUpdateBy;
	}

	public Date getLastUpdateTime() {
		return lastUpdateTime;
	}

	public void setLastUpdateTime(Date lastUpdateTime) {
		this.lastUpdateTime = lastUpdateTime;
	}
    
}

```

SysUser

```java
package com.louis.kitty.admin.model;

import java.util.ArrayList;
import java.util.List;

public class SysUser extends BaseModel {

    private String name;

    private String password;

    private String salt;

    private String email;

    private String mobile;

    private Byte status;

    private Long deptId;
    
    private String deptName;
    
    private Byte delFlag;
    
    private String roleNames;
    
    private List<SysUserRole> userRoles = new ArrayList<>();

	public String getName() {
		return name;
	}

	public void setName(String name) {
		this.name = name;
	}

	public String getPassword() {
		return password;
	}

	public void setPassword(String password) {
		this.password = password;
	}

	public String getSalt() {
		return salt;
	}

	public void setSalt(String salt) {
		this.salt = salt;
	}

	public String getEmail() {
		return email;
	}

	public void setEmail(String email) {
		this.email = email;
	}

	public String getMobile() {
		return mobile;
	}

	public void setMobile(String mobile) {
		this.mobile = mobile;
	}

	public Byte getStatus() {
		return status;
	}

	public void setStatus(Byte status) {
		this.status = status;
	}

	public Long getDeptId() {
		return deptId;
	}

	public void setDeptId(Long deptId) {
		this.deptId = deptId;
	}

	public String getDeptName() {
		return deptName;
	}

	public void setDeptName(String deptName) {
		this.deptName = deptName;
	}

	public Byte getDelFlag() {
		return delFlag;
	}

	public void setDelFlag(Byte delFlag) {
		this.delFlag = delFlag;
	}

	public String getRoleNames() {
		return roleNames;
	}

	public void setRoleNames(String roleNames) {
		this.roleNames = roleNames;
	}

	public List<SysUserRole> getUserRoles() {
		return userRoles;
	}

	public void setUserRoles(List<SysUserRole> userRoles) {
		this.userRoles = userRoles;
	}

}
```

### 持久层

为了更好的让熟悉它，此处模拟了一个自定义的SQL，可以发现使用 `通用Mapper` 后并不会破坏原有代码结构

> UserMapper

继承 `BaseMapper<T>` 就可以了，这点是不是有点类似 `JpaRepository`，同时也可以根据自己需要扩展出更适合自己项目的`BaseMapper`，它的灵活也是众多开发者喜爱的因素之一

```java
package com.louis.kitty.admin.dao;

import java.util.List;

import org.apache.ibatis.annotations.Param;

import com.louis.kitty.admin.model.SysUser;

public interface SysUserMapper {
    int deleteByPrimaryKey(Long id);

    int insert(SysUser record);

    int insertSelective(SysUser record);

    SysUser selectByPrimaryKey(Long id);

    int updateByPrimaryKeySelective(SysUser record);

    int updateByPrimaryKey(SysUser record);
    
    List<SysUser> findPage();
    
    SysUser findByName(@Param(value="name") String name);
    
	List<SysUser> findPageByName(@Param(value="name") String name);
}
```

> `UserMapper` 映射文件

SysUserMapper.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.louis.kitty.admin.dao.SysUserMapper">
  <resultMap id="BaseResultMap" type="com.louis.kitty.admin.model.SysUser">
    <id column="id" jdbcType="BIGINT" property="id" />
    <result column="name" jdbcType="VARCHAR" property="name" />
    <result column="password" jdbcType="VARCHAR" property="password" />
    <result column="salt" jdbcType="VARCHAR" property="salt" />
    <result column="email" jdbcType="VARCHAR" property="email" />
    <result column="mobile" jdbcType="VARCHAR" property="mobile" />
    <result column="status" jdbcType="TINYINT" property="status" />
    <result column="dept_id" jdbcType="BIGINT" property="deptId" />
    <result column="create_by" jdbcType="BIGINT" property="createBy" />
    <result column="create_time" jdbcType="TIMESTAMP" property="createTime" />
    <result column="last_update_by" jdbcType="BIGINT" property="lastUpdateBy" />
    <result column="last_update_time" jdbcType="TIMESTAMP" property="lastUpdateTime" />
    <result column="del_flag" jdbcType="TINYINT" property="delFlag" />
  </resultMap>
  <sql id="Base_Column_List">
    id, name, password, salt, email, mobile, status, dept_id, create_by, create_time, 
    last_update_by, last_update_time, del_flag
  </sql>
  <select id="selectByPrimaryKey" parameterType="java.lang.Long" resultMap="BaseResultMap">
    select 
    <include refid="Base_Column_List" />
    from sys_user
    where id = #{id,jdbcType=BIGINT}
  </select>
  <delete id="deleteByPrimaryKey" parameterType="java.lang.Long">
    delete from sys_user
    where id = #{id,jdbcType=BIGINT}
  </delete>
  <insert id="insert" parameterType="com.louis.kitty.admin.model.SysUser">
    insert into sys_user (id, name, password, 
      salt, email, mobile, 
      status, dept_id, create_by, 
      create_time, last_update_by, last_update_time, 
      del_flag)
    values (#{id,jdbcType=BIGINT}, #{name,jdbcType=VARCHAR}, #{password,jdbcType=VARCHAR}, 
      #{salt,jdbcType=VARCHAR}, #{email,jdbcType=VARCHAR}, #{mobile,jdbcType=VARCHAR}, 
      #{status,jdbcType=TINYINT}, #{deptId,jdbcType=BIGINT}, #{createBy,jdbcType=BIGINT}, 
      #{createTime,jdbcType=TIMESTAMP}, #{lastUpdateBy,jdbcType=BIGINT}, #{lastUpdateTime,jdbcType=TIMESTAMP}, 
      #{delFlag,jdbcType=TINYINT})
  </insert>
  <insert id="insertSelective" parameterType="com.louis.kitty.admin.model.SysUser">
    insert into sys_user
    <trim prefix="(" suffix=")" suffixOverrides=",">
      <if test="id != null">
        id,
      </if>
      <if test="name != null">
        name,
      </if>
      <if test="password != null">
        password,
      </if>
      <if test="salt != null">
        salt,
      </if>
      <if test="email != null">
        email,
      </if>
      <if test="mobile != null">
        mobile,
      </if>
      <if test="status != null">
        status,
      </if>
      <if test="deptId != null">
        dept_id,
      </if>
      <if test="createBy != null">
        create_by,
      </if>
      <if test="createTime != null">
        create_time,
      </if>
      <if test="lastUpdateBy != null">
        last_update_by,
      </if>
      <if test="lastUpdateTime != null">
        last_update_time,
      </if>
      <if test="delFlag != null">
        del_flag,
      </if>
    </trim>
    <trim prefix="values (" suffix=")" suffixOverrides=",">
      <if test="id != null">
        #{id,jdbcType=BIGINT},
      </if>
      <if test="name != null">
        #{name,jdbcType=VARCHAR},
      </if>
      <if test="password != null">
        #{password,jdbcType=VARCHAR},
      </if>
      <if test="salt != null">
        #{salt,jdbcType=VARCHAR},
      </if>
      <if test="email != null">
        #{email,jdbcType=VARCHAR},
      </if>
      <if test="mobile != null">
        #{mobile,jdbcType=VARCHAR},
      </if>
      <if test="status != null">
        #{status,jdbcType=TINYINT},
      </if>
      <if test="deptId != null">
        #{deptId,jdbcType=BIGINT},
      </if>
      <if test="createBy != null">
        #{createBy,jdbcType=BIGINT},
      </if>
      <if test="createTime != null">
        #{createTime,jdbcType=TIMESTAMP},
      </if>
      <if test="lastUpdateBy != null">
        #{lastUpdateBy,jdbcType=BIGINT},
      </if>
      <if test="lastUpdateTime != null">
        #{lastUpdateTime,jdbcType=TIMESTAMP},
      </if>
      <if test="delFlag != null">
        #{delFlag,jdbcType=TINYINT},
      </if>
    </trim>
  </insert>
  <update id="updateByPrimaryKeySelective" parameterType="com.louis.kitty.admin.model.SysUser">
    update sys_user
    <set>
      <if test="name != null">
        name = #{name,jdbcType=VARCHAR},
      </if>
      <if test="password != null">
        password = #{password,jdbcType=VARCHAR},
      </if>
      <if test="salt != null">
        salt = #{salt,jdbcType=VARCHAR},
      </if>
      <if test="email != null">
        email = #{email,jdbcType=VARCHAR},
      </if>
      <if test="mobile != null">
        mobile = #{mobile,jdbcType=VARCHAR},
      </if>
      <if test="status != null">
        status = #{status,jdbcType=TINYINT},
      </if>
      <if test="deptId != null">
        dept_id = #{deptId,jdbcType=BIGINT},
      </if>
      <if test="createBy != null">
        create_by = #{createBy,jdbcType=BIGINT},
      </if>
      <if test="createTime != null">
        create_time = #{createTime,jdbcType=TIMESTAMP},
      </if>
      <if test="lastUpdateBy != null">
        last_update_by = #{lastUpdateBy,jdbcType=BIGINT},
      </if>
      <if test="lastUpdateTime != null">
        last_update_time = #{lastUpdateTime,jdbcType=TIMESTAMP},
      </if>
      <if test="delFlag != null">
        del_flag = #{delFlag,jdbcType=TINYINT},
      </if>
    </set>
    where id = #{id,jdbcType=BIGINT}
  </update>
  <update id="updateByPrimaryKey" parameterType="com.louis.kitty.admin.model.SysUser">
    update sys_user
    set name = #{name,jdbcType=VARCHAR},
      password = #{password,jdbcType=VARCHAR},
      salt = #{salt,jdbcType=VARCHAR},
      email = #{email,jdbcType=VARCHAR},
      mobile = #{mobile,jdbcType=VARCHAR},
      status = #{status,jdbcType=TINYINT},
      dept_id = #{deptId,jdbcType=BIGINT},
      create_by = #{createBy,jdbcType=BIGINT},
      create_time = #{createTime,jdbcType=TIMESTAMP},
      last_update_by = #{lastUpdateBy,jdbcType=BIGINT},
      last_update_time = #{lastUpdateTime,jdbcType=TIMESTAMP},
      del_flag = #{delFlag,jdbcType=TINYINT}
    where id = #{id,jdbcType=BIGINT}
  </update>
  <select id="findPage" resultMap="BaseResultMap">
    select u.*, (select d.name from sys_dept d where d.id = u.dept_id) deptName from sys_user u
  </select>
  <select id="findByName" parameterType="java.lang.String" resultMap="BaseResultMap">
    select u.*, (select d.name from sys_dept d where d.id = u.dept_id) deptName from sys_user u
    where u.name = #{name,jdbcType=VARCHAR}
  </select>
  <select id="findPageByName" parameterType="java.lang.String" resultMap="BaseResultMap">
  	<bind name="pattern" value="'%' + _parameter.name + '%'" />
    select u.*, (select d.name from sys_dept d where d.id = u.dept_id) deptName from sys_user u
    where u.name like #{pattern}
  </select>
</mapper>
```

### 服务层和服务接口层

```java
package com.louis.kitty.admin.service;

import java.util.List;
import java.util.Set;

import com.louis.kitty.admin.model.SysUser;
import com.louis.kitty.admin.model.SysUserRole;
import com.louis.kitty.core.service.CurdService;

public interface SysUserService extends CurdService<SysUser>{


    SysUser findByName(String username);

    /**
     * 查找用户的菜单权限标识集合
     * @param userName
     * @return
     */
    Set<String> findPermissions(String userName);

    /**
     * 查找用户的角色集合
     * @param userName
     * @return
     */
    List<SysUserRole> findUserRoles(Long userId);

}

```



```java
package com.louis.kitty.admin.service.impl;

import java.util.HashSet;
import java.util.Iterator;
import java.util.List;
import java.util.Set;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import com.louis.kitty.admin.config.ColumnFilter;
import com.louis.kitty.admin.dao.SysRoleMapper;
import com.louis.kitty.admin.dao.SysUserMapper;
import com.louis.kitty.admin.dao.SysUserRoleMapper;
import com.louis.kitty.admin.model.SysMenu;
import com.louis.kitty.admin.model.SysRole;
import com.louis.kitty.admin.model.SysUser;
import com.louis.kitty.admin.model.SysUserRole;
import com.louis.kitty.admin.page.MybatisPageHelper;
import com.louis.kitty.admin.page.PageRequest;
import com.louis.kitty.admin.page.PageResult;
import com.louis.kitty.admin.service.SysMenuService;
import com.louis.kitty.admin.service.SysUserService;

@Service
public class SysUserServiceImpl  implements SysUserService {

        @Autowired
        private SysUserMapper sysUserMapper;
        @Autowired
        private SysMenuService sysMenuService;
        @Autowired
        private SysUserRoleMapper sysUserRoleMapper;
        @Autowired
        private SysRoleMapper sysRoleMapper;

        @Transactional
        @Override
        public int save(SysUser record) {
                Long id = null;
                if(record.getId() == null || record.getId() == 0) {
                        // 新增用户
                        sysUserMapper.insertSelective(record);
                        id = record.getId();
                } else {
                        // 更新用户信息
                        sysUserMapper.updateByPrimaryKeySelective(record);
                }
                // 更新用户角色
                if(id != null && id == 0) {
                        return 1;
                }
                if(id != null) {
                        for(SysUserRole sysUserRole:record.getUserRoles()) {
                                sysUserRole.setUserId(id);
                        }
                } else {
                        sysUserRoleMapper.deleteByUserId(record.getId());
                }
                for(SysUserRole sysUserRole:record.getUserRoles()) {
                        sysUserRoleMapper.insertSelective(sysUserRole);
                }
                return 1;
        }

        @Override
        public int delete(SysUser record) {
                return sysUserMapper.deleteByPrimaryKey(record.getId());
        }

        @Override
        public int delete(List<SysUser> records) {
                for(SysUser record:records) {
                        delete(record);
                }
                return 1;
        }

        @Override
        public SysUser findById(Long id) {
                return sysUserMapper.selectByPrimaryKey(id);
        }
        
        @Override
        public SysUser findByName(String name) {
                return sysUserMapper.findByName(name);
        }
        
        @Override
        public PageResult findPage(PageRequest pageRequest) {
                PageResult pageResult = null;
                ColumnFilter columnFilter = pageRequest.getColumnFilter("name");
                if(columnFilter != null && columnFilter.getValue() != null) {
                        pageResult = MybatisPageHelper.findPage(pageRequest, sysUserMapper, "findPageByName", columnFilter.getValue());
                } else {
                        pageResult = MybatisPageHelper.findPage(pageRequest, sysUserMapper);
                }
                // 加载用户角色信息
                findUserRoles(pageResult);
                return pageResult;
        }

        /**
         * 加载用户角色
         * @param pageResult
         */
        private void findUserRoles(PageResult pageResult) {
                List<?> content = pageResult.getContent();
                for(Object object:content) {
                        SysUser sysUser = (SysUser) object;
                        List<SysUserRole> userRoles = findUserRoles(sysUser.getId());
                        sysUser.setUserRoles(userRoles);
                        sysUser.setRoleNames(getRoleNames(userRoles));
                }
        }

        private String getRoleNames(List<SysUserRole> userRoles) {
                StringBuilder sb = new StringBuilder();
                for(Iterator<SysUserRole> iter=userRoles.iterator(); iter.hasNext();) {
                        SysUserRole userRole = iter.next();
                        SysRole sysRole = sysRoleMapper.selectByPrimaryKey(userRole.getRoleId());
                        if(sysRole == null) {
                                continue ;
                        }
                        sb.append(sysRole.getRemark());
                        if(iter.hasNext()) {
                                sb.append(", ");
                        }
                }
                return sb.toString();
        }

        @Override
        public Set<String> findPermissions(String userName) {   
                Set<String> perms = new HashSet<>();
                List<SysMenu> sysMenus = sysMenuService.findByUser(userName);
                for(SysMenu sysMenu:sysMenus) {
                        perms.add(sysMenu.getPerms());
                }
                return perms;
        }

        @Override
        public List<SysUserRole> findUserRoles(Long userId) {
                return sysUserRoleMapper.findUserRoles(userId);
        }
}
```

### 控制层

接口入口访问控制

```java
package com.louis.kitty.admin.controller;

import java.util.List;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

import com.louis.kitty.admin.constants.SysConstants;
import com.louis.kitty.admin.model.SysUser;
import com.louis.kitty.admin.page.PageRequest;
import com.louis.kitty.admin.result.HttpResult;
import com.louis.kitty.admin.service.SysUserService;
import com.louis.kitty.admin.utils.PasswordUtils;
import com.louis.kitty.admin.utils.ShiroUtils;

/**
 * 用户控制器
 * @author Louis
 * @date Oct 29, 2018
 */
@RestController
@RequestMapping("user")
public class SysUserController {

	@Autowired
	private SysUserService sysUserService;
	
	@PostMapping(value="/save")
	public HttpResult save(@RequestBody SysUser record) {
		SysUser user = sysUserService.findById(record.getId());
		if(user != null) {
			if(SysConstants.ADMIN.equalsIgnoreCase(user.getName())) {
				return HttpResult.error("超级管理员不允许修改!");
			}
		}
		if(record.getPassword() != null) {
			String salt = PasswordUtils.getSalt();
			if(user == null) {
				// 新增用户
				if(sysUserService.findByName(record.getName()) != null) {
					return HttpResult.error("用户名已存在!");
				}
				String password = PasswordUtils.encrypte(record.getPassword(), salt);
				record.setSalt(salt);
				record.setPassword(password);
			} else {
				// 修改用户, 且修改了密码
				if(!record.getPassword().equals(user.getPassword())) {
					String password = PasswordUtils.encrypte(record.getPassword(), salt);
					record.setSalt(salt);
					record.setPassword(password);
				}
			}
		}
		return HttpResult.ok(sysUserService.save(record));
	}

	@PostMapping(value="/delete")
	public HttpResult delete(@RequestBody List<SysUser> records) {
		for(SysUser record:records) {
			SysUser sysUser = sysUserService.findById(record.getId());
			if(sysUser != null && SysConstants.ADMIN.equalsIgnoreCase(sysUser.getName())) {
				return HttpResult.error("超级管理员不允许删除!");
			}
		}
		return HttpResult.ok(sysUserService.delete(records));
	}
	
	@GetMapping(value="/findByName")
	public HttpResult findByUserName(@RequestParam String name) {
		return HttpResult.ok(sysUserService.findByName(name));
	}
	
	@GetMapping(value="/findPermissions")
	public HttpResult findPermissions(@RequestParam String name) {
		return HttpResult.ok(sysUserService.findPermissions(name));
	}
	
	@GetMapping(value="/findUserRoles")
	public HttpResult findUserRoles(@RequestParam Long userId) {
		return HttpResult.ok(sysUserService.findUserRoles(userId));
	}

	@PostMapping(value="/findPage")
	public HttpResult findPage(@RequestBody PageRequest pageRequest) {
		return HttpResult.ok(sysUserService.findPage(pageRequest));
	}
	
	/**
	 * 修改登录用户密码
	 */
	@GetMapping("/updatePassword")
	public HttpResult updatePassword(@RequestParam String password, @RequestParam String newPassword) {
		SysUser user = ShiroUtils.getUser();
		if(user != null && password != null && newPassword != null) {
			String oldPassword = PasswordUtils.encrypte(password, user.getSalt());
			if(!oldPassword.equals(user.getPassword())) {
				return HttpResult.error("原密码不正确");
			}
			user.setPassword(PasswordUtils.encrypte(newPassword, user.getSalt()));
			HttpResult.ok(sysUserService.save(user));
		}
		return HttpResult.error();
	}

}

```

## 总结

1. **Mybatis官方文档：** <http://www.mybatis.org/mybatis-3/zh/index.html>
2. **通用Mapper文档：** <https://gitee.com/free/Mapper/wikis/1.1-java?parent=1.integration>
3. **分页插件文档：** <https://github.com/pagehelper/Mybatis-PageHelper/blob/master/wikis/zh/HowToUse.md>

   4.**参考文章：**

​	①**如何优雅的使用mybatis:**<http://www.ityouknow.com/springboot/2016/11/06/spring-boo-mybatis.html>

​	②**整合Mybatis:**<https://blog.battcn.com/2018/05/09/springboot/v2-orm-mybatis/>

​	③**通用Mapper与分页插件的集成：**<https://blog.battcn.com/2018/05/10/springboot/v2-orm-mybatis-plugin/>