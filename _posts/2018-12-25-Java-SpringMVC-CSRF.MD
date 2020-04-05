---
title:  "SpringMVC拦截器防御CSRF攻击"
categories: SpringMVC
tags: SpringMVC
author: LIUREN
---

# Spring mvc拦截器防御CSRF攻击

>  CSRF（Cross-site request forgery跨站请求伪造，也被称为“One Click Attack”或者Session Riding，通常缩写为CSRF或者XSRF，是一种对网站的恶意利用。尽管听起来像跨站脚本（XSS），但它与XSS非常不同，并且攻击方式几乎相左。XSS利用站点内的信任用户，而CSRF则通过伪装来自受信任用户的请求来利用受信任的网站。与XSS攻击相比，CSRF攻击往往不大流行（因此对其进行防范的资源也相当稀少）和难以防范，所以被认为比XSS更具危险性



具体思路：

1、跳转页面前生成随机token，并存放在session中

2、form中将token放在隐藏域中，保存时将token放头部一起提交

3、获取头部token，与session中的token比较，一致则通过，否则不予提交

4、生成新的token，并传给前端

代码：

1、拦截器配置

```xml
	<mvc:interceptors>
	    <!-- csrf攻击防御 -->
		<mvc:interceptor>
			<!-- 需拦截的地址 -->
			<mvc:mapping path="/**" />
			<!-- 需排除拦截的地址 -->
			<mvc:exclude-mapping path="/resources/**" />
			<bean class="com.cnpc.framework.interceptor.CSRFInterceptor" />
		</mvc:interceptor>
	</mvc:interceptors>
```

2、拦截器实现 CSRFInterceptor.java

```java
package com.cnpc.framework.interceptor;
 
import java.io.OutputStream;
import java.io.PrintWriter;
import java.lang.reflect.Method;
import java.util.HashMap;
import java.util.Map;
 
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
 
import org.springframework.web.method.HandlerMethod;
import org.springframework.web.servlet.ModelAndView;
import org.springframework.web.servlet.handler.HandlerInterceptorAdapter;
 
import com.alibaba.fastjson.JSONObject;
import com.cnpc.framework.annotation.RefreshCSRFToken;
import com.cnpc.framework.annotation.VerifyCSRFToken;
import com.cnpc.framework.base.pojo.ResultCode;
import com.cnpc.framework.constant.CodeConstant;
import com.cnpc.framework.utils.CSRFTokenUtil;
import com.cnpc.framework.utils.StrUtil;
 
/**
 * CSRFInterceptor 防止跨站请求伪造拦截器
 *
 * @author billJiang 2016年10月6日 下午8:14:40
 */
public class CSRFInterceptor extends HandlerInterceptorAdapter {
 
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
 
        System.out.println("---------->" + request.getRequestURI());
        System.out.println(request.getHeader("X-Requested-With"));
        // 提交表单token 校验
        HandlerMethod handlerMethod = (HandlerMethod) handler;
        Method method = handlerMethod.getMethod();
        VerifyCSRFToken verifyCSRFToken = method.getAnnotation(VerifyCSRFToken.class);
        // 如果配置了校验csrf token校验，则校验
        if (verifyCSRFToken != null) {
            // 是否为Ajax标志
            String xrq = request.getHeader("X-Requested-With");
            // 非法的跨站请求校验
            if (verifyCSRFToken.verify() && !verifyCSRFToken(request)) {
                if (StrUtil.isEmpty(xrq)) {
                    // form表单提交，url get方式，刷新csrftoken并跳转提示页面
                    String csrftoken = CSRFTokenUtil.generate(request);
                    request.getSession().setAttribute("CSRFToken", csrftoken);
                    response.setContentType("application/json;charset=UTF-8");
                    PrintWriter out = response.getWriter();
                    out.print("非法请求");
                    response.flushBuffer();
                    return false;
                } else {
                    // 刷新CSRFToken，返回错误码，用于ajax处理，可自定义
                    String csrftoken = CSRFTokenUtil.generate(request);
                    request.getSession().setAttribute("CSRFToken", csrftoken);
                    ResultCode rc = CodeConstant.CSRF_ERROR;
                    response.setContentType("application/json;charset=UTF-8");
                    PrintWriter out = response.getWriter();
                    out.print(JSONObject.toJSONString(rc));
                    response.flushBuffer();
                    return false;
                }
            }
 
        }
        return true;
    }
 
    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView)
            throws Exception {
 
        // 第一次生成token
        if (modelAndView != null) {
            if (request.getSession(false) == null || StrUtil.isEmpty((String) request.getSession(false).getAttribute("CSRFToken"))) {
                request.getSession().setAttribute("CSRFToken", CSRFTokenUtil.generate(request));
                return;
            }
        }
        // 刷新token
        HandlerMethod handlerMethod = (HandlerMethod) handler;
        Method method = handlerMethod.getMethod();
        RefreshCSRFToken refreshAnnotation = method.getAnnotation(RefreshCSRFToken.class);
 
        // 跳转到一个新页面 刷新token
        String xrq = request.getHeader("X-Requested-With");
        if (refreshAnnotation != null && refreshAnnotation.refresh() && StrUtil.isEmpty(xrq)) {
            request.getSession().setAttribute("CSRFToken", CSRFTokenUtil.generate(request));
            return;
        }
 
        // 校验成功 刷新token 可以防止重复提交
        VerifyCSRFToken verifyAnnotation = method.getAnnotation(VerifyCSRFToken.class);
        if (verifyAnnotation != null) {
            if (verifyAnnotation.verify()) {
                if (StrUtil.isEmpty(xrq)) {
                    request.getSession().setAttribute("CSRFToken", CSRFTokenUtil.generate(request));
                } else {
                    Map<String, String> map = new HashMap<String, String>();
                    map.put("CSRFToken", CSRFTokenUtil.generate(request));
                    response.setContentType("application/json;charset=UTF-8");
                    OutputStream out = response.getOutputStream();
                    out.write((",'csrf':" + JSONObject.toJSONString(map) + "}").getBytes("UTF-8"));
                }
            }
        }
    }
 
    /**
     * 处理跨站请求伪造 针对需要登录后才能处理的请求,验证CSRFToken校验
     * 
     * @param request
     */
    protected boolean verifyCSRFToken(HttpServletRequest request) {
 
        // 请求中的CSRFToken
        String requstCSRFToken = request.getHeader("CSRFToken");
        if (StrUtil.isEmpty(requstCSRFToken)) {
            return false;
        }
        String sessionCSRFToken = (String) request.getSession().getAttribute("CSRFToken");
        if (StrUtil.isEmpty(sessionCSRFToken)) {
            return false;
        }
        return requstCSRFToken.equals(sessionCSRFToken);
    }
}
```

3、CSRFTokenUtil.java

```java
package com.cnpc.framework.utils;
 
import java.util.UUID;
 
import javax.servlet.http.HttpServletRequest;
 
public class CSRFTokenUtil {
 
    public static String generate(HttpServletRequest request) {
 
        return UUID.randomUUID().toString();
    }
 
}
```

4、ResultCode

```java
package com.cnpc.framework.base.pojo;
 
public class ResultCode {
 
    private String code;
 
    private String message;
 
    public ResultCode(String code, String message) {
 
        this.code = code;
        this.message = message;
    }
 
    public String getCode() {
 
        return code;
    }
 
    public void setCode(String code) {
 
        this.code = code;
    }
 
    public String getMessage() {
 
        return message;
    }
 
    public void setMessage(String message) {
 
        this.message = message;
    }
 
}
```

5、CodeConstant.java

```java
package com.cnpc.framework.constant;
 
import com.cnpc.framework.base.pojo.ResultCode;
 
public class CodeConstant {
 
    public final static ResultCode CSRF_ERROR = new ResultCode("101", "CSRF ERROR:无效的token，或者token过期");
}
```

6、ajax提交方法

```js
function ajaxPost(url, params, callback) {
	var result = null;
    var headers={};
    headers['CSRFToken']=$("#csrftoken").val();
    
	$.ajax({
		type : 'post',
		async : false,
		url : url,
		data : params,
		dataType : 'json',
		headers:headers,
		success : function(data, status) {
			result = data;
			if(data&&data.code&&data.code=='101'){
				modals.error("操作失败，请刷新重试，具体错误："+data.message);
				return false;
			}
			if (callback) { 
				callback.call(this, data, status);
			}
		},
		error : function(err, err1, err2) {
		    if(err && err.readyState && err.readyState == '4'){
                var responseBody = err.responseText;
                if(responseBody){   
                	 responseBody = "{'retData':"+responseBody;
                     var resJson = eval('(' + responseBody + ')');
                     $("#csrftoken").val(resJson.csrf.CSRFToken);
                     this.success(resJson.retData, 200);
                }
                return ;
            } 		    
			modals.error({
				text : JSON.stringify(err) + '<br/>err1:' + JSON.stringify(err1) + '<br/>err2:' + JSON.stringify(err2),
				large : true
			});
		}
	});
 
	return result;
}
```

7、form表单配置

```html
<input type='hidden' value='${CSRFToken}' id='csrftoken'>
```

8、注解定义

```java
package com.cnpc.framework.annotation;
 
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;
 
/**
 * 跨站请求仿照注解 刷新CSRFToken
 * 
 */
@Target({ java.lang.annotation.ElementType.METHOD })
@Retention(RetentionPolicy.RUNTIME)
public @interface RefreshCSRFToken {
 
    /**
     * 刷新token
     *
     * @return
     */
    public abstract boolean refresh() default true;
}
```

```java
package com.cnpc.framework.annotation;
 
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;
 
/**
 * 跨站请求仿照注解
 */
@Target({ java.lang.annotation.ElementType.METHOD })
@Retention(RetentionPolicy.RUNTIME)
public @interface VerifyCSRFToken {
 
    /**
     * 需要验证防跨站请求
     * 
     */
    public abstract boolean verify() default true;
 
}
```

9、注解配置

```java
   @RefreshCSRFToken
    @RequestMapping(method = RequestMethod.GET, value = "/edit")
    private String edit(String id, HttpServletRequest request) {
 
        request.setAttribute("id", id);
        return "base/user/user_edit";
    }
 
    @RequestMapping(method = RequestMethod.POST, value = "/get")
    @ResponseBody
    private User getUser(String id) {
 
        return userService.get(User.class, id);
    }
```



> 

