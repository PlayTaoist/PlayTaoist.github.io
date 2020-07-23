---
title:  "使用SpringBoot使用过滤器去除@RequestBody参数两端的空格"
categories: SpringBoot RequestBody
tags: SpringBoot RequestBody
author: LIUREN
---

# 使用SpringBoot使用过滤器去除@RequestBody参数两端的空格

> 使用SpringBoot使用过滤器去除@RequestBody参数两端的空格；一般我们去普通的请求我们都会对请求参数进行验证。Java也提供了@notNull和@notBlank这种验证方式，但是对@RequestBody 这种只能验证是不是非空，对数据两端的空格未进行处理，同时大家也不想遍历一遍参数然后再处理再封装到对象中，正好项目中有这个需要，所以就参考别的做了Post请求中针对application/json格式的有@RequestBody注解的参数进行了去空格处理
>
> 协议：CC BY-SA 4.0 https://creativecommons.org/licenses/by-sa/4.0/  
>
> 版权声明：本文为原创文章，遵循 CC 4.0 BY-SA 版权协议，转载请附上原文出处链接及本声明。



## 第一步：编写一个配置信息

ParamsFilterConfig.java

```java
package com.codepeople.framework.config;

import javax.servlet.DispatcherType;

import org.springframework.boot.web.servlet.FilterRegistrationBean;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import com.ruoyi.framework.filter.ParamsFilter;

/**
 * @ClassName: ParamsFilterConfig
 * @Description: SpringBoot中去除@RequestBody中前后端空格
 * @Author 刘 仁
 * @DateTime 2020-7-23 10:07:02
 */
@Configuration
public class ParamsFilterConfig {

	@Bean
	public FilterRegistrationBean paramsFilterRegistration() {
		FilterRegistrationBean registration = new FilterRegistrationBean();
		registration.setDispatcherTypes(DispatcherType.REQUEST);
        registration.setFilter(new ParamsFilter());
        registration.addUrlPatterns("/*");
        registration.setName("paramsFilter");
        registration.setOrder(Integer.MAX_VALUE-1);
        return registration;
	}
}
```



## 第二步：编写ParamsFilter过滤器

ParamsFilter.java

```java
package com.codepeople.framework.filter;

import java.io.IOException;

import javax.servlet.DispatcherType;
import javax.servlet.Filter;
import javax.servlet.FilterChain;
import javax.servlet.FilterConfig;
import javax.servlet.ServletException;
import javax.servlet.ServletRequest;
import javax.servlet.ServletResponse;
import javax.servlet.annotation.WebFilter;
import javax.servlet.http.HttpServletRequest;

import org.springframework.stereotype.Component;

@Component
@WebFilter(urlPatterns = "/**", filterName = "ParamsFilter", dispatcherTypes = DispatcherType.REQUEST)
public class ParamsFilter implements Filter {

	@Override
	public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain)
			throws IOException, ServletException {
		ParameterRequestWrapper parmsRequest = new ParameterRequestWrapper((HttpServletRequest) request);
		chain.doFilter(parmsRequest, response);
	}

	@Override
	public void init(FilterConfig filterConfig) throws ServletException {
		
	}

	@Override
	public void destroy() {
		
	}

	
}
```

## 第三步：实现ParameterRequestWrapper

ParameterRequestWrapper.java

```java
package com.codepeople.framework.filter;

import java.io.ByteArrayInputStream;
import java.io.IOException;
import java.util.HashMap;
import java.util.Iterator;
import java.util.Map;
import java.util.Set;

import javax.servlet.ReadListener;
import javax.servlet.ServletInputStream;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletRequestWrapper;

import org.apache.commons.io.IOUtils;
import org.apache.commons.lang3.StringUtils;
import org.springframework.http.HttpHeaders;
import org.springframework.http.MediaType;

import com.alibaba.fastjson.JSON;
import com.codepeople.framework.util.StringJsonUtils;

public class ParameterRequestWrapper extends HttpServletRequestWrapper {

	private Map<String , String[]> params = new HashMap<>();
	
	public ParameterRequestWrapper(HttpServletRequest request) {
		// 将request交给父类，以便于调用对应方法的时候，将其输出，其实父亲类的实现方式和第一种new的方式类似
        super(request);
        //将参数表，赋予给当前的Map以便于持有request中的参数
        Map<String, String[]> requestMap=request.getParameterMap();
        this.params.putAll(requestMap);
        this.modifyParameterValues();
	}

	/**
	 * 重写getInputStream方法  post类型的请求参数必须通过流才能获取到值 
	 */
	@Override
    public ServletInputStream getInputStream() throws IOException {
		/** 非json类型，直接返回 */
		if(!super.getHeader(HttpHeaders.CONTENT_TYPE).equalsIgnoreCase(MediaType.APPLICATION_JSON_VALUE)){
            return super.getInputStream();
        }
        //为空，直接返回
        String json = IOUtils.toString(super.getInputStream(), "utf-8");
        if (StringUtils.isEmpty(json)) {
            return super.getInputStream();
        }
        System.out.println("去除POST请求数据两端的空格前参数："+json);
        Map<String,Object> map= StringJsonUtils.jsonStringToMap(json);
        System.out.println("去除POST请求数据两端的空格后参数："+JSON.toJSONString(map));
        ByteArrayInputStream bis = new ByteArrayInputStream(JSON.toJSONString(map).getBytes("utf-8"));
        return new MyServletInputStream(bis);
    }

	/**
	 * @Title: modifyParameterValues
	 * @Description:  将parameter的值去除空格后重写回去
	 * @DateTime 2020-7-23 10:31:12
	 */
    public void modifyParameterValues(){
        Set<String> set = params.keySet();
        Iterator<String> it = set.iterator();
        while(it.hasNext()){
            String key= it.next();
            String[] values = params.get(key);
            values[0] = values[0].trim();
            params.put(key, values);
        }
    }
    
    @Override
    public String getParameter(String name) {
        String[]values = params.get(name);
        if(values == null || values.length == 0) {
            return null;
        }
        return values[0];
    }
    /**
	 * @Title: getParameterValues
	 * @Description:  重写getParameterValues
	 * @DateTime 2020-7-23 10:31:12
	 */
    @Override
    public String[] getParameterValues(String name) {//同上
        return params.get(name);
    }
 
    class MyServletInputStream extends  ServletInputStream{
        private ByteArrayInputStream bis;
        public MyServletInputStream(ByteArrayInputStream bis){
            this.bis=bis;
        }
        @Override
        public boolean isFinished() {
            return true;
        }
 
        @Override
        public boolean isReady() {
            return true;
        }
 
        @Override
        public void setReadListener(ReadListener listener) {
 
        }
        @Override
        public int read(){
            return bis.read();
        }
    }

}
```

## 第四步：利用fastjson实现json字符串转map功能

```java
package com.codepeople.framework.util;

import java.util.ArrayList;
import java.util.HashMap;
import java.util.Iterator;
import java.util.List;
import java.util.Map;

import com.alibaba.fastjson.JSONArray;
import com.alibaba.fastjson.JSONObject;
/**
 * @ClassName: StringJsonUtils
 * @Description: 
 * @Author 刘 仁
 * @DateTime 2020-7-23 10:26:28
 */
public class StringJsonUtils {
	
	/**
	 * @Title: jsonStringToMap
	 * @Description: 把jsonString转为Map
	 * @Author 刘 仁
	 * @DateTime 2020-7-23 10:27:11
	 * @param jsonString
	 * @return
	 */
	public static Map<String, Object> jsonStringToMap(String jsonString) {
        Map<String, Object> map = new HashMap<>();
        JSONObject jsonObject = JSONObject.parseObject(jsonString);
        for (Object k : jsonObject.keySet()) {
            Object o = jsonObject.get(k);
            if (o instanceof JSONArray) {
                List<Map<String, Object>> list = new ArrayList<>();
                Iterator<Object> it = ((JSONArray) o).iterator();
                while (it.hasNext()) {
                    Object obj = it.next();
                    list.add(jsonStringToMap(obj.toString()));
                }
                map.put(k.toString(), list);
            } else if (o instanceof JSONObject) {
                // 如果内层是json对象的话，继续解析
                map.put(k.toString(), jsonStringToMap(o.toString()));
            } else {
                // 如果内层是普通对象的话，直接放入map中
               // map.put(k.toString(), o.toString().trim()); 
                 if (o instanceof String) {
                   map.put(k.toString(), o.toString().trim());
                } else {
                    map.put(k.toString(), o);
                }   
            }
        }
        return map;
    }

}
```

主要引入的包

```xml
<!--常用工具类 -->
		<dependency>
			<groupId>org.apache.commons</groupId>
			<artifactId>commons-lang3</artifactId>
		</dependency>
<!-- 阿里JSON解析器 -->
		<dependency>
			<groupId>com.alibaba</groupId>
			<artifactId>fastjson</artifactId>
		</dependency>
<!-- io常用工具类 -->
		<dependency>
			<groupId>commons-io</groupId>
			<artifactId>commons-io</artifactId>
		</dependency>
<!-- servlet包 -->
        <dependency>
		    <groupId>javax.servlet</groupId>
		    <artifactId>javax.servlet-api</artifactId>
		</dependency>
```

测试日志

```shell
11:28:07.995 [http-nio-80-exec-17] INFO  c.r.f.f.ParameterRequestWrapper - [getInputStream,54] - 去除POST请求数据两端的空格前参数：{
	"idNumber": "110101201003073036",
	"password": " 123456 "
}
11:28:07.997 [http-nio-80-exec-17] INFO  c.r.f.f.ParameterRequestWrapper - [getInputStream,56] - 去除POST请求数据两端的空格后参数：{"password":"123456","idNumber":"110101201003073036"}
```



博客地址：<https://www.codepeople.cn>

=====================================================================

微信公众号：

![](https://www.codepeople.cn/imges/weixin_icon/weixin.jpg)