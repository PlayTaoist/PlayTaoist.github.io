---
title:  "防止同一IP多次请求攻击"
categories: Springboot2.x
tags: Springboot2.x
author: LIUREN
---

# 防止同一IP多次请求攻击

> 防止入侵者，通过死循环同一时间批量向服务器请求数据，导致服务器内存开销不断膨胀，最后直接瘫痪。



## 一、 新增一个spring的拦截器 ， 拦截所有请求



```powershell
    <mvc:interceptor>  
            <!-- 拦截所有请求，判断是否多次请求 -->  
            <mvc:mapping path="/*/*" />
            <bean class="com.boyu.interceptor.RequestManageInterceptor" />  
        </mvc:interceptor> 
```

拦截器实现 HandlerInterceptor 接口，重写preHandle、postHandle、afterCompletion方法。

在preHandle方法里面

```java
        PrintWriter out = null;//返回给页面显示
        Map<String,Object> resultMap = new HashMap<String,Object>();
                //取用户的真实IP
        String ip  =  request.getHeader("x-forwarded-for");  

        if (ip == null || ip.length() == 0 || " unknown ".equalsIgnoreCase(ip)) {
            ip = request.getHeader(" Proxy-Client-IP ");
        }
        if (ip == null || ip.length() == 0 || " unknown ".equalsIgnoreCase(ip)) {
            ip = request.getHeader(" WL-Proxy-Client-IP ");
        }
        if (ip == null || ip.length() == 0 || " unknown ".equalsIgnoreCase(ip)) {
            ip = request.getRemoteAddr();
        }
               //取session中的IP对象
               RequestIp re = (RequestIp) request.getSession().getAttribute(ip);
               //第一次请求
        if(null == re){
            //放入到session中
            RequestIp reIp = new RequestIp();
            reIp.setCreateTime(System.currentTimeMillis());
            reIp.setReCount(1);
            request.getSession().setAttribute(ip,reIp);
        }else{
            Long createTime = re.getCreateTime();
            if(null == createTime){
                //时间请求为空
                resultMap.put("code", 503);
                resultMap.put("message", "请求太快，请稍后再试！");
                out = response.getWriter();
                out.append(ResultUtil.resultMapToString(resultMap));
            }else{
                if(((System.currentTimeMillis() - createTime)/1000) > 3){
                    System.out.println("通过请求！"+((System.currentTimeMillis() - createTime)/1000));
                    //当前时间离上一次请求时间大于3秒，可以直接通过,保存这次的请求
                    RequestIp reIp = new RequestIp();
                    reIp.setCreateTime(System.currentTimeMillis());
                    reIp.setReCount(1);
                    request.getSession().setAttribute(ip,reIp);
                }else{
                                        //小于3秒，并且3秒之内请求了10次，返回提示
                    if(re.getReCount() > 10){
                        resultMap.put("code", 503);
                        resultMap.put("message", "请求太快，请稍后再试！");
                        out = response.getWriter();
                        out.append(ResultUtil.resultMapToString(resultMap));//以json形式返回给页面，也可以直接返回提示信息
                        return false;
                    }else{
                                        //小于3秒，但请求数小于10次，给对象添加
                        re.setCreateTime(System.currentTimeMillis());
                        re.setReCount(re.getReCount()+1);
                        request.getSession().setAttribute(ip,re);
                    }
                }
            }
        } 
```

RequestIp.java 

```java
private String ip ;
private long createTime;
private Integer reCount;
```



原文链接：<https://blog.csdn.net/mr__su/article/details/51604863>

=====================================================================

博客地址：<https://www.codepeople.cn>

=====================================================================

微信公众号：

![](https://www.codepeople.cn/imges/weixin_icon/weixin.jpg)