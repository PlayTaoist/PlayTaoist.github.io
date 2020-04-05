---
title:  "SpringBoot2.0跨域配置"
categories: SpringBoot2.0 Shiro
tags: SpringBoot2.0 Shiro
author: LIUREN
---

# SpringBoot2.0跨域配置

> ## 什么是跨域？
>
> 同源策略是浏览器的一个安全功能，不同源的客户端脚本在没有明确授权的情况下，不能读写对方资源。 同源策略是浏览器安全的基石。
>
> 如果一个请求地址里面的协议、域名和端口号都相同，就属于同源。
>
> 举个栗子
>
> 判断下面URL是否和 http://www.a.com/a/a.html 同源
>
> - http://www.a.com/b/b.html 同源
> - http://www.b.com/a/a.html 不同源，域名不相同
> - https://www.a.com/b/b.html 不同源，协议不相同
> - http://www.a.com:8080/b/b.html 不同源，端口号不相同
>
> 依据浏览器同源策略，非同源脚本不可操作其他源下面的对象。想要操作其他源下的对象就需要跨域。
>
> 综上所述，在同源策略的限制下，非同源的网站之间不能发送 `AJAX` 请求。如有需要，可通过降域或其他技术实现。



## CORS 技术

为了解决浏览器跨域问题，`W3C` 提出了跨源资源共享方案，即 `CORS`([Cross-Origin Resource Sharing](https://www.w3.org/TR/cors/))。

`CORS` 可以在不破坏即有规则的情况下，通过后端服务器实现 `CORS` 接口，就可以实现跨域通信。

`CORS` 将请求分为两类：简单请求和非简单请求，分别对跨域通信提供了支持。

### 1、简单请求

在`CORS`出现前，发送`HTTP`请求时在头信息中不能包含任何自定义字段，且 `HTTP` 头信息不超过以下几个字段：

- `Accept`
- `Accept-Language`
- `Content-Language`
- `Last-Event-ID`
- `Content-Type` （仅限于 [`application/x-www-form-urlencoded` 、`multipart/form-data`、`text/plain` ] 类型）

一个简单请求的例子：

```
GET /test HTTP/1.1
Accept: */*
Accept-Encoding: gzip, deflate, sdch, br
Origin: http://www.test.com
Host: www.test.com
```

对于简单请求，`CORS`的策略是请求时在请求头中增加一个`Origin`字段，服务器收到请求后，根据该字段判断是否允许该请求访问。

- 如果允许，则在 HTTP 头信息中添加 `Access-Control-Allow-Origin` 字段，并返回正确的结果 ；
- 如果不允许，则不在 HTTP 头信息中添加 `Access-Control-Allow-Origin` 字段 。

除了上面提到的 `Access-Control-Allow-Origin` ，还有几个字段用于描述 `CORS` 返回结果 ：

- `Access-Control-Allow-Credentials`： 可选，用户是否可以发送、处理 `cookie`；
- `Access-Control-Expose-Headers`：可选，可以让用户拿到的字段。有几个字段无论设置与否都可以拿到的，包括：`Cache-Control`、`Content-Language`、`Content-Type`、`Expires`、`Last-Modified`、`Pragma` 。

### 2、非简单请求

对于非简单请求的跨源请求，浏览器会在真实请求发出前，增加一次`OPTION`请求，称为预检请求(`preflight request`)。预检请求将真实请求的信息，包括请求方法、自定义头字段、源信息添加到 HTTP 头信息字段中，询问服务器是否允许这样的操作。

例如一个`GET`请求：

```
OPTIONS /test HTTP/1.1
Origin: http://www.test.com
Access-Control-Request-Method: GET
Access-Control-Request-Headers: X-Custom-Header
Host: www.test.com
```

与 `CORS` 相关的字段有：

- 请求使用的 `HTTP` 方法 `Access-Control-Request-Method` 
- 请求中包含的自定义头字段 `Access-Control-Request-Headers` 

服务器收到请求时，需要分别对 `Origin`、`Access-Control-Request-Method`、`Access-Control-Request-Headers` 进行验证，验证通过后，会在返回 `HTTP`头信息中添加 ：

```
Access-Control-Allow-Origin: http://www.test.com
Access-Control-Allow-Methods: GET, POST, PUT, DELETE
Access-Control-Allow-Headers: X-Custom-Header
Access-Control-Allow-Credentials: true
Access-Control-Max-Age: 1728000
```

他们的含义分别是：

- Access-Control-Allow-Methods: 真实请求允许的方法
- Access-Control-Allow-Headers: 服务器允许使用的字段
- Access-Control-Allow-Credentials: 是否允许用户发送、处理 cookie
- Access-Control-Max-Age: 预检请求的有效期，单位为秒。有效期内，不会重复发送预检请求

当预检请求通过后，浏览器才会发送真实请求到服务器。这样就实现了跨域资源的请求访问。



## 项目添加跨域支持

增加配置文件CorsConfig.java

```java
package com.louis.kitty.boot.config;

import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.CorsRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;

@Configuration
public class CorsConfig implements WebMvcConfigurer {

    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/**")    // 允许跨域访问的路径
        .allowedOrigins("*")    // 允许跨域访问的源
        .allowedMethods("POST", "GET", "PUT", "OPTIONS", "DELETE")    // 允许请求方法
        .maxAge(168000)    // 预检间隔时间
        .allowedHeaders("*")  // 允许头部设置
        .allowCredentials(true);    // 是否发送cookie
    }
}
```



### 2.修改过滤器

#### 2.1 Shiro 导致的跨域问题

按照正常逻辑，添加了上面的跨域配置类就可以实现跨域支持了。然而，我们使用了 Shiro 就不一样了。我们上面讲到，对于非简单的跨域请求，会事先发起一个OPTION类型的预检请求，只有预检请求成功才会发起真正的请求，而这个预检请求是不带 token 的，这就意味着这个预检请求会被 shiro 过滤器拦截并在 token 校验失败之后返回失败信息，从而不会再发起真正的请求。

![](https://www.codepeople.cn/imges/cors/001.png)



#### 2.2 跨域解决方案

解决思路很简单，既然是因为预检请求失败导致的问题，那就让预检请求自动放行就可以了。

![](https://www.codepeople.cn/imges/cors/002.png)

==================================================================

[博客地址](https://www.codepeople.cn)：<https://www.codepeople.cn>

==================================================================

微信公众号：

![](https://www.codepeople.cn/imges/weixin_icon/weixin.jpg)