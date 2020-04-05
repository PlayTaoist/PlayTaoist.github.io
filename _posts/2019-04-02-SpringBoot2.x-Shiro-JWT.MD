---
title:  "SpringBoot2.0+Shiro+JWT 整合"
categories: SpringBoot2.0 Shiro JWT
tags: SpringBoot2.0 Shiro JWT
author: LIUREN
---

# SpringBoot2.0+Shiro+JWT 整合

> JSON Web Token（JWT）是一个非常轻巧的规范。这个规范允许我们使用 JWT 在用户和服务器之间传递安全可靠的信息。
> 我们利用一定的编码生成 Token，并在 Token 中加入一些非敏感信息，将其传递。



## 安装环境

```shell
开发工具：STS
Maven版本：apache-maven-3.5.2
java jdk 1.8
MySQL版本：5.7
系统：Windows10
```



## 一.新建Maven项目

配置Maven项目的pom.xml文件

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.1.3.RELEASE</version>
        <relativePath /> <!-- lookup parent from repository -->
    </parent>
    
  <groupId>cn.codepeople</groupId>
  <artifactId>SpringBoot-Shiro-JWT</artifactId>
  <version>0.0.1-SNAPSHOT</version>
  
  <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <java.version>1.8</java.version>
        <log4j.version>1.2.17</log4j.version>
        <shiro.version>1.4.0</shiro.version>
        <com.alibaba.druid.version>1.1.10</com.alibaba.druid.version>
        <mysql.version>5.1.47</mysql.version>
        <mybatis.version>2.0.0</mybatis.version>
        <jwt.version>3.8.0</jwt.version>
        <swagger2.version>2.8.0</swagger2.version>
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
        <!-- Apache Commons -->
        <dependency>
            <groupId>commons-collections</groupId>
            <artifactId>commons-collections</artifactId>
            <version>3.2.2</version>
        </dependency>
        <dependency>
            <groupId>commons-lang</groupId>
            <artifactId>commons-lang</artifactId>
            <version>2.6</version>
        </dependency>
        <dependency>
            <groupId>org.apache.commons</groupId>
            <artifactId>commons-lang3</artifactId>
        </dependency>
        <!-- https://mvnrepository.com/artifact/com.alibaba/fastjson -->
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>fastjson</artifactId>
            <version>1.2.54</version>
        </dependency>
        <!-- https://mvnrepository.com/artifact/com.auth0/java-jwt -->
        <dependency>
            <groupId>com.auth0</groupId>
            <artifactId>java-jwt</artifactId>
            <version>${jwt.version}</version>
        </dependency>
        <!-- swagger -->
        <dependency>
            <groupId>io.springfox</groupId>
            <artifactId>springfox-swagger2</artifactId>
            <version>${swagger2.version}</version>
        </dependency>
        <dependency>
            <groupId>io.springfox</groupId>
            <artifactId>springfox-swagger-ui</artifactId>
            <version>${swagger2.version}</version>
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



## 二.准备shiro和jwt工具和相关类

JWTFilter.java

```java
/**
 * @Title: JWTFilter.java
 * @Package www.codepeople.cn.shiro
 * @Description: 
 * Copyright: Copyright (c) 2019 www.codepeople.cn Inc. All rights reserved. 
 * Website: www.codepeople.cn
 * @Author 刘仁
 * @DateTime 2019年4月1日 下午4:52:19
 * @version V1.0
 */

package www.codepeople.cn.shiro;

import java.io.IOException;

import javax.servlet.ServletRequest;
import javax.servlet.ServletResponse;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

import org.apache.shiro.web.filter.authc.BasicHttpAuthenticationFilter;
import org.springframework.http.HttpStatus;
import org.springframework.web.bind.annotation.RequestMethod;

import lombok.extern.slf4j.Slf4j;

/**
 * @ClassName: JWTFilter
 * @Description: 
 * @Author 刘仁
 * @DateTime 2019年4月1日 下午4:52:19 
 */
@Slf4j
public class JWTFilter extends BasicHttpAuthenticationFilter{

	/**
     * 判断用户是否想要登入。
     * 检测header里面是否包含Authorization字段即可
     */
    @Override
    protected boolean isLoginAttempt(ServletRequest request, ServletResponse response) {
        HttpServletRequest req = (HttpServletRequest) request;
        String authorization = req.getHeader("Authorization");
        log.info("判断用户是否想要登录：{}",authorization);
        return authorization != null;
    }

    /**
     *
     */
    @Override
    protected boolean executeLogin(ServletRequest request, ServletResponse response) throws Exception {
        HttpServletRequest httpServletRequest = (HttpServletRequest) request;
        String authorization = httpServletRequest.getHeader("Authorization");
        log.info("判断用户是否想要登录x：{}",authorization);
        JWTToken token = new JWTToken(authorization);
        // 提交给realm进行登入，如果错误他会抛出异常并被捕获
        getSubject(request, response).login(token);
        // 如果没有抛出异常则代表登入成功，返回true
        return true;
    }

    /**
     * 这里我们详细说明下为什么最终返回的都是true，即允许访问
     * 例如我们提供一个地址 GET /article
     * 登入用户和游客看到的内容是不同的
     * 如果在这里返回了false，请求会被直接拦截，用户看不到任何东西
     * 所以我们在这里返回true，Controller中可以通过 subject.isAuthenticated() 来判断用户是否登入
     * 如果有些资源只有登入用户才能访问，我们只需要在方法上面加上 @RequiresAuthentication 注解即可
     * 但是这样做有一个缺点，就是不能够对GET,POST等请求进行分别过滤鉴权(因为我们重写了官方的方法)，但实际上对应用影响不大
     */
    @Override
    protected boolean isAccessAllowed(ServletRequest request, ServletResponse response, Object mappedValue) {
        if (isLoginAttempt(request, response)) {
            try {
                executeLogin(request, response);
            } catch (Exception e) {
                response401(request, response);
            }
        }
        return true;
    }

    /**
     * 对跨域提供支持
     */
    @Override
    protected boolean preHandle(ServletRequest request, ServletResponse response) throws Exception {
        HttpServletRequest httpServletRequest = (HttpServletRequest) request;
        HttpServletResponse httpServletResponse = (HttpServletResponse) response;
        httpServletResponse.setHeader("Access-control-Allow-Origin", httpServletRequest.getHeader("Origin"));
        httpServletResponse.setHeader("Access-Control-Allow-Methods", "GET,POST,OPTIONS,PUT,DELETE");
        httpServletResponse.setHeader("Access-Control-Allow-Headers", httpServletRequest.getHeader("Access-Control-Request-Headers"));
        // 跨域时会首先发送一个option请求，这里我们给option请求直接返回正常状态
        if (httpServletRequest.getMethod().equals(RequestMethod.OPTIONS.name())) {
            httpServletResponse.setStatus(HttpStatus.OK.value());
            return false;
        }
        return super.preHandle(request, response);
    }

    /**
     * 将非法请求跳转到 /401
     */
    private void response401(ServletRequest req, ServletResponse resp) {
        try {
            HttpServletResponse httpServletResponse = (HttpServletResponse) resp;
            httpServletResponse.sendRedirect("/401");
        } catch (IOException e) {
            log.error(e.getMessage());
        }
    }
}
```

JWTToken.java

```java
/**
 * @Title: JWTToken.java
 * @Package www.codepeople.cn.shiro
 * @Description: 
 * Copyright: Copyright (c) 2019 www.codepeople.cn Inc. All rights reserved. 
 * Website: www.codepeople.cn
 * @Author 刘仁
 * @DateTime 2019年4月1日 下午4:49:43
 * @version V1.0
 */

package www.codepeople.cn.shiro;

import org.apache.shiro.authc.AuthenticationToken;

/**
 * @ClassName: JWTToken
 * @Description: 
 * @Author 刘仁
 * @DateTime 2019年4月1日 下午4:49:43 
 */

public class JWTToken implements AuthenticationToken {
	
	private static final long serialVersionUID = 1L;
	// 秘钥
	private String token;
	
	public JWTToken(String token) {
		this.token = token;
	}
	@Override
	public Object getPrincipal() {
		return token;
	}

	
	@Override
	public Object getCredentials() {
		return token;
	}

}
```

MyRealm.java

```java
/**
 * @Title: MyRealm.java
 * @Package www.codepeople.cn.shiro
 * @Description: 
 * Copyright: Copyright (c) 2019 www.codepeople.cn Inc. All rights reserved. 
 * Website: www.codepeople.cn
 * @Author 刘仁
 * @DateTime 2019年4月1日 下午4:31:33
 * @version V1.0
 */

package www.codepeople.cn.shiro;

import java.util.Arrays;
import java.util.HashSet;
import java.util.Set;

import org.apache.shiro.authc.AuthenticationException;
import org.apache.shiro.authc.AuthenticationInfo;
import org.apache.shiro.authc.AuthenticationToken;
import org.apache.shiro.authc.SimpleAuthenticationInfo;
import org.apache.shiro.authz.AuthorizationInfo;
import org.apache.shiro.authz.SimpleAuthorizationInfo;
import org.apache.shiro.realm.AuthorizingRealm;
import org.apache.shiro.subject.PrincipalCollection;
import org.springframework.beans.factory.annotation.Autowired;

import www.codepeople.cn.entity.User;
import www.codepeople.cn.service.UserService;
import www.codepeople.cn.util.JWTUtil;

/**
 * @ClassName: MyRealm
 * @Description:
 * @Author 刘仁
 * @DateTime 2019年4月1日 下午4:31:33
 */

public class MyRealm extends AuthorizingRealm {

	@Autowired
	private UserService userService;
	
	@Override
	public boolean supports(AuthenticationToken token) {
		return token instanceof JWTToken;
	}

	@Override
	protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principals) {
		String username = JWTUtil.getUsername(principals.toString());
		User user = userService.getUser(username);
		SimpleAuthorizationInfo simpleAuthorizationInfo = new SimpleAuthorizationInfo();
		simpleAuthorizationInfo.addRole(user.getRole());
		Set<String> permission = new HashSet<String>(Arrays.asList(user.getPerms().split(",")));
		simpleAuthorizationInfo.addStringPermissions(permission);
		
		return simpleAuthorizationInfo;
	}

	/**
	 * 默认使用此方法进行用户正确与否验证，错误抛出异常即可
	 */
	@Override
	protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken authenticationToken) throws AuthenticationException {
		String token = (String) authenticationToken.getCredentials();
		// 解密获得username，用于和数据库进行对比
		String username = JWTUtil.getUsername(token);
		if (username == null) {
			throw new AuthenticationException("token 无效！");
		}
		
		User user = userService.getUser(username);
		if (user == null) {
			throw new AuthenticationException("用户"+username+"不存在") ;
		}
		
		if (!JWTUtil.verify(token, username, user.getPassword())) {
			throw new AuthenticationException("账户密码错误!");
		}
		return new SimpleAuthenticationInfo(token, token, "my_realm");
	}

}
```

ShiroConfig.java

```java
/**
 * @Title: ShiroConfig.java
 * @Package www.codepeople.cn.shiro
 * @Description: 
 * Copyright: Copyright (c) 2019 www.codepeople.cn Inc. All rights reserved. 
 * Website: www.codepeople.cn
 * @Author 刘仁
 * @DateTime 2019年4月1日 下午4:14:32
 * @version V1.0
 */

package www.codepeople.cn.shiro;

import java.util.HashMap;
import java.util.LinkedHashMap;
import java.util.Map;

import javax.servlet.Filter;

import org.apache.shiro.mgt.DefaultSecurityManager;
import org.apache.shiro.spring.LifecycleBeanPostProcessor;
import org.apache.shiro.spring.security.interceptor.AuthorizationAttributeSourceAdvisor;
import org.apache.shiro.spring.web.ShiroFilterFactoryBean;
import org.apache.shiro.web.mgt.DefaultWebSecurityManager;
import org.springframework.aop.framework.autoproxy.DefaultAdvisorAutoProxyCreator;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.DependsOn;

/**
 * @ClassName: ShiroConfig
 * @Description: 
 * @Author 刘仁
 * @DateTime 2019年4月1日 下午4:14:32 
 */
@Configuration
public class ShiroConfig {

	@Bean
	public ShiroFilterFactoryBean getShiroFilterFactoryBean(@Qualifier("securityManager") DefaultSecurityManager securityManager) {
		ShiroFilterFactoryBean shiroFilterFactoryBean = new ShiroFilterFactoryBean();
		
		// 添加自己的过滤器并且取名为jwt
		Map<String, Filter> filterMap = new HashMap<>();
		filterMap.put("jwt", new JWTFilter());
		shiroFilterFactoryBean.setFilters(filterMap);
		shiroFilterFactoryBean.setSecurityManager(securityManager);
		shiroFilterFactoryBean.setUnauthorizedUrl("/401");
		/**
         * 自定义url规则
         * http://shiro.apache.org/web.html#urls-
         */
		Map<String, String> filterRuleMap = new LinkedHashMap<String, String>();
		// 所有的请求通过我们自己的JWT filter
		filterRuleMap.put("/**", "jwt");
		// 访问401和404页面不通过我们的Filter
		filterRuleMap.put("/401", "anon");
		filterRuleMap.put("/404", "anon");
		
		shiroFilterFactoryBean.setFilterChainDefinitionMap(filterRuleMap);
		return shiroFilterFactoryBean;
	}
	
	@Bean(name = "securityManager")
	public DefaultSecurityManager getDefaultSecurityManager(@Qualifier("myRelam") MyRealm myRealm) {
		DefaultWebSecurityManager securityManager = new DefaultWebSecurityManager();
		securityManager.setRealm(myRealm);
		return securityManager;
	}
	
	@Bean(name="myRelam")
	public MyRealm getMyRealm() {
		return new MyRealm();
	}
	
	 /**
     * 下面的代码是添加注解支持
     */
    @Bean
    @DependsOn("lifecycleBeanPostProcessor")
    public DefaultAdvisorAutoProxyCreator defaultAdvisorAutoProxyCreator() {
        DefaultAdvisorAutoProxyCreator defaultAdvisorAutoProxyCreator = new DefaultAdvisorAutoProxyCreator();
        // 强制使用cglib，防止重复代理和可能引起代理出错的问题
        // https://zhuanlan.zhihu.com/p/29161098
        defaultAdvisorAutoProxyCreator.setProxyTargetClass(true);
        return defaultAdvisorAutoProxyCreator;
    }

    @Bean
    public LifecycleBeanPostProcessor lifecycleBeanPostProcessor() {
        return new LifecycleBeanPostProcessor();
    }

    @Bean
    public AuthorizationAttributeSourceAdvisor authorizationAttributeSourceAdvisor(DefaultWebSecurityManager securityManager) {
        AuthorizationAttributeSourceAdvisor advisor = new AuthorizationAttributeSourceAdvisor();
        advisor.setSecurityManager(securityManager);
        return advisor;
    }
}
```

JWTUtil.java

```java
/**
 * @Title: JWTUtil.java
 * @Package www.codepeople.cn.util
 * @Description: 
 * Copyright: Copyright (c) 2019 www.codepeople.cn Inc. All rights reserved. 
 * Website: www.codepeople.cn
 * @Author 刘仁
 * @DateTime 2019年4月1日 下午4:32:56
 * @version V1.0
 */

package www.codepeople.cn.util;

import java.io.UnsupportedEncodingException;
import java.util.Date;

import com.auth0.jwt.JWT;
import com.auth0.jwt.JWTVerifier;
import com.auth0.jwt.algorithms.Algorithm;
import com.auth0.jwt.exceptions.JWTDecodeException;
import com.auth0.jwt.interfaces.DecodedJWT;

/**
 * @ClassName: JWTUtil
 * @Description: 
 * @Author 刘仁
 * @DateTime 2019年4月1日 下午4:32:56 
 */

public class JWTUtil {
	// 过期时间24小时
	private static final long EXPRIE_TIME = 24*60*60*1000;
	
	public static boolean verify(String token, String username, String secret) {
		try {
			Algorithm algorithm = Algorithm.HMAC512(secret);
			JWTVerifier verifier = JWT.require(algorithm)
					.withClaim("username", username)
					.build();
			verifier.verify(token);
			return true;
		} catch (Exception e) {
			return false;
		}
	}
	/**
	 * @Title: getUsername
	 * @Description: 获取token中的信息无需secret解密也能获得
	 * @Author 刘仁
	 * @DateTime 2019年4月1日 下午4:42:39
	 * @param token
	 * @return
	 */
	public static String getUsername(String token) {
		try {
			DecodedJWT jwt = JWT.decode(token);
			return jwt.getClaim("username").toString();
		} catch (JWTDecodeException e) {
			return null;
		}
	}
	
	public static String sign(String username, String secret) throws UnsupportedEncodingException {
		Date date = new Date(System.currentTimeMillis()+EXPRIE_TIME);
		Algorithm algorithm = Algorithm.HMAC512(secret);
		// 附带username信息
		return JWT.create()
				.withClaim("username", username)
				.withExpiresAt(date)
				.sign(algorithm);
	}
}
```

登录功能实现

UserController.java

```java
/**
 * @Title: UserController.java
 * @Package www.codepeople.cn.controller
 * @Description: 
 * Copyright: Copyright (c) 2019 www.codepeople.cn Inc. All rights reserved. 
 * Website: www.codepeople.cn 
 * @Author 刘仁
 * @DateTime 2019年4月1日 下午5:29:03
 * @version V1.0
 */

package www.codepeople.cn.controller;

import java.io.UnsupportedEncodingException;
import java.util.List;

import javax.servlet.http.HttpServletResponse;

import org.apache.shiro.authz.UnauthorizedException;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.HttpStatus;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.ResponseStatus;
import org.springframework.web.bind.annotation.RestController;

import io.swagger.annotations.ApiImplicitParam;
import io.swagger.annotations.ApiImplicitParams;
import io.swagger.annotations.ApiOperation;
import www.codepeople.cn.entity.ResponseBean;
import www.codepeople.cn.entity.User;
import www.codepeople.cn.service.UserService;
import www.codepeople.cn.util.JWTUtil;

/**
 * @ClassName: UserController
 * @Description: 
 * @Author 刘仁
 * @DateTime 2019年4月1日 下午5:29:03 
 */
@RestController
public class UserController {

	@Autowired
	private UserService userService;
	
	@PostMapping("/login")
	@ApiOperation(value="登录功能", notes="用户密码登录")
	@ApiImplicitParams({
         @ApiImplicitParam(name = "username", value = "用户名", required = true, dataType = "String"),
         @ApiImplicitParam(name = "password", value = "密码", required = true, dataType = "String")
	 })
	public ResponseBean login(@RequestParam("username") String username,
							  @RequestParam("password") String password,
							  HttpServletResponse response) throws UnsupportedEncodingException {
		User user = userService.getUser(username);
		if (user.getPassword().equals(password)) {
			String token = JWTUtil.sign(username, password);
			response.setHeader("token", token);
			return new ResponseBean(200, "登录成功", token);
		} else {
			throw new UnauthorizedException();
		}
	}
	
	@GetMapping("/listUser")
	@ApiOperation(value="获取用户列表", notes="获取用户列表")
	public ResponseBean listUser() throws UnsupportedEncodingException {
		List<User> listUser = userService.listUser();
		return new ResponseBean(200, "用户列表", listUser);
	}
	
	@RequestMapping("/401")
	@ResponseStatus(HttpStatus.UNAUTHORIZED)
	public ResponseBean unauthorized() {
		return new ResponseBean(401, "未授权", null);
	}
}
```

为了方便调试加入了Swagger的架包

整体的代码下载地址：<https://gitee.com/VCS/springboot-shiro-jwt-demo>

==================================================================

[博客地址](https://www.codepeople.cn)：<https://www.codepeople.cn>

==================================================================

微信公众号：

![](https://www.codepeople.cn/imges/weixin_icon/weixin.jpg)