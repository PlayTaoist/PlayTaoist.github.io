---
title:  "SpringBoot2.0中Long类型丢失精度问题"
categories: SpringBoot2.0
tags: SpringBoot2.0
author: LIUREN
---

# SpringBoot2.0中Long类型丢失精度问题

> 在使用框架过程中，发现如果ID超过17位，Long就会发生精度丢失问题，精度丢失不是在Java端，而是在js端。针对此问题有如下解决方案



## 安装环境

```shell
开发工具：STS
Maven版本：apache-maven-3.5.2
java jdk 1.8
MySQL版本：5.7
```



## 一.重新编写jackson转换器

```java
package io.renren.common.config;

import java.math.BigInteger;
import java.nio.charset.Charset;
import java.util.List;

import org.springframework.context.annotation.Configuration;
import org.springframework.http.converter.HttpMessageConverter;
import org.springframework.http.converter.StringHttpMessageConverter;
import org.springframework.http.converter.json.MappingJackson2HttpMessageConverter;
import org.springframework.web.servlet.config.annotation.EnableWebMvc;
import org.springframework.web.servlet.config.annotation.ResourceHandlerRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;

import com.fasterxml.jackson.databind.ObjectMapper;
import com.fasterxml.jackson.databind.module.SimpleModule;
import com.fasterxml.jackson.databind.ser.std.ToStringSerializer;
/**
 * WebMvc配置
 *
 * @author LiuRen
 */
@Configuration
@EnableWebMvc
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void addResourceHandlers(ResourceHandlerRegistry registry) {
        registry.addResourceHandler("/statics/**").addResourceLocations("classpath:/statics/");
    }
    
    /**
     * * 针对Long丢失经度问题；发现后台返回前台的数据不会出现问题。前台请求的数据也是正常的
     * * 只不过前台js获取到页面上会出现问题，针对以上情况，在前台js中，修改id的name值为id_
     */
    @Override
    public void configureMessageConverters(List<HttpMessageConverter<?>> converters) {
        MappingJackson2HttpMessageConverter jackson2HttpMessageConverter =
                new MappingJackson2HttpMessageConverter();
        
        ObjectMapper objectMapper = new ObjectMapper();
        SimpleModule simpleModule = new SimpleModule();
        simpleModule.addSerializer(BigInteger.class, ToStringSerializer.instance);
        simpleModule.addSerializer(Long.class, ToStringSerializer.instance);
        simpleModule.addSerializer(Long.TYPE, ToStringSerializer.instance);
        objectMapper.registerModule(simpleModule);
        jackson2HttpMessageConverter.setObjectMapper(objectMapper);
        converters.add(jackson2HttpMessageConverter);
        converters.add(new StringHttpMessageConverter(Charset.forName("UTF-8")));
    }
    

}
```



如果使用以上方法，则会直接把Long类型的数据转换为String类型，从而可以防止Long类型精度丢失问题

## 二.注意问题

**注意:** 但是中间会产生问题，对Date类型的数据，配置信息中会失效，如果需要保持时间类型的数据不变化，最好的解决方案是在 `实体类`的上面加上`@JsonFormat(timezone = "GMT+8", pattern = "yyyy-MM-dd HH:mm:ss")` 的注解，例如：

```java
package io.renren.modules.soft.entity;

import java.io.Serializable;
import java.util.Date;

import com.baomidou.mybatisplus.annotation.TableId;
import com.baomidou.mybatisplus.annotation.TableLogic;
import com.fasterxml.jackson.annotation.JsonFormat;

import lombok.Data;

@Data
public class BaseEntity implements Serializable{

	private static final long serialVersionUID = 1L;

	@TableId
	private Long id;
	
	/**
	 * 创建时间
	 */
	@JsonFormat(timezone = "GMT+8", pattern = "yyyy-MM-dd HH:mm:ss")
	private Date createTime;
	/**
	 * 创建人
	 */
	private String createBy;
	/**
	 * 更新时间
	 */
	@JsonFormat(timezone = "GMT+8", pattern = "yyyy-MM-dd HH:mm:ss")
	private Date updateTime;
	/**
	 * 更新人
	 */
	private String updateBy;
	/**
	 * 删除状态
	 */
	@TableLogic
	private String status;
}
```



==================================================================

[博客地址](https://www.codepeople.cn)：<https://www.codepeople.cn>

==================================================================

微信公众号：

![](https://www.codepeople.cn/imges/weixin_icon/weixin.jpg)