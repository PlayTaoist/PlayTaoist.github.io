---
title:  "SpringBoot2.0支持跨域请求全局配置和局部配置"
categories: SpringBoot2.0
tags: SpringBoot2.0
author: LIUREN
---

# SpringBoot2.0支持跨域请求全局配置和局部配置

> 本文记录了如何使用SpringBoot2.0支持跨域请求全局配置和局部配置。



## SpringBoot2.0支持跨域请求全局配置和局部配置

1.编写一个支持跨域请求的 Configuration

## - 第一种方式

### - CorsConfig.java

```java
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.CorsRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurerAdapter;

/**
 * 处理AJAX请求跨域的问题
 * @author Levin
 * @time 2017-07-13
 */
@Configuration
public class CorsConfig extends WebMvcConfigurerAdapter {
	static final String ORIGINS[] = new String[] { "GET", "POST", "PUT", "DELETE" };
	@Override
	public void addCorsMappings(CorsRegistry registry) {
        //设置允许跨域的路径
		registry.addMapping("/**")
            	//设置允许跨域请求的域名
            	.allowedOrigins("*")
            	//是否允许证书 不再默认开启
            	.allowCredentials(true)
            	//设置允许的方法
            	.allowedMethods(ORIGINS)
            	//跨域允许时间
				.maxAge(3600);
	}
}
```

2.HTTP请求接口

### - HelloController.java

```java
@RestController
public class HelloController {
	
	@Autowired
	HelloService helloService;
	
	@GetMapping(value = "/test", produces = MediaType.APPLICATION_JSON_UTF8_VALUE)
	public String query() {
		return "hello";
	}
}
```



## - 第二种方式（推荐）

PS：第一种存在一个问题，当服务器抛出 500 的时候依旧存在跨域问题



```java
@SpringBootApplication
@ComponentScan
@EnableDiscoveryClient
public class ManagementApplication {

    public static void main(String[] args) {
        SpringApplication.run(ManagementApplication.class, args);
    }

    private CorsConfiguration buildConfig() {
        CorsConfiguration corsConfiguration = new CorsConfiguration();
        corsConfiguration.addAllowedOrigin("*");
        corsConfiguration.addAllowedHeader("*");
        corsConfiguration.addAllowedMethod("*");
        corsConfiguration.addExposedHeader(HttpHeaderConStant.X_TOTAL_COUNT);
        return corsConfiguration;
    }

    /**
     * 跨域过滤器
     *
     * @return
     */
    @Bean
    public CorsFilter corsFilter() {
        UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
        source.registerCorsConfiguration("/**", buildConfig()); // 4
        return new CorsFilter(source);
    }
}
```

## - index.html

```html
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<title>跨域请求</title>
<script src="https://cdn.bootcss.com/jquery/1.10.2/jquery.min.js">
</script>
<script>
$(document).ready(function(){
	$("button").click(function(){
		$.ajax({url:"http://localhost:8080/test",success:function(result){
			$("#p1").html(result);
		}});
	});
});
</script>
</head>
<body>

<p width="500px" height="100px" id="p1"></p>
<button>获取其他内容</button>
</body>
</html>
```

## - 第三种方式

主要使用@CrossOrigin 注解，全局的配置 在@CrossOrigin里依旧可用

可以注解在单个方法上

```java
@RestController
@RequestMapping(/account)
public class AccountController {
 
    @CrossOrigin
    @GetMapping(/{id})
    public Account retrieve(@PathVariable Long id) {
        
    }
 
    @DeleteMapping(/{id})
    public void remove(@PathVariable Long id) {
        
    }
}

```

也可以注解在整个controller上

```java
@CrossOrigin(origins = http://domain2.com, maxAge = 3600)
@RestController
@RequestMapping(/account)
public class AccountController {
 
    @GetMapping(/{id})
    public Account retrieve(@PathVariable Long id) {
        
    }
 
    @DeleteMapping(/{id})
    public void remove(@PathVariable Long id) {
        
    }
}

```

也可以解在整个controller上的同时注解在单个方法上

```java
@CrossOrigin(maxAge = 3600)
@RestController
@RequestMapping(/account)
public class AccountController {
 
    @CrossOrigin(http://domain2.com)
    @GetMapping(/{id})
    public Account retrieve(@PathVariable Long id) {
        
    }
 
    @DeleteMapping(/{id})
    public void remove(@PathVariable Long id) {
        
    }
}

```



# 设置白名单

以上配置，默认所有其他服务都可以发起跨域请求，存在安全问题，修改修改可以设定只允许指定ip跨域请求

```java
		registry.addMapping("/**")
			.allowedOrigins("http://192.168.112.1:8081");
```

注意：此时在浏览器用localhost请求html是不能跨域的，因为这里配的是ip，所以只认ip域名，参数中可以配多个ip。

