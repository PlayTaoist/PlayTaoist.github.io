---
title:  "修复Long类型太长，而Java序列化JSON丢失精度问题的方法"
categories: SpringBoot
tags: SpringBoot
author: LIUREN
---

# 修复Long类型太长，而Java序列化JSON丢失精度问题的方法

> 修复Long类型太长，而Java序列化JSON丢失精度问题的方法,主要学习了两家之长Spring的Jackson和FastJson两种序列化Long转String的两种方案，顺便做一下笔记
> 作者：liuren
>
> 版权声明：本文为原创文章，遵循 CC 4.0 BY-SA 版权协议，转载请附上原文出处链接及本声明。



最近在给前端返回Long类型的数据时，发现返回的数据到前端的时候，精度发生了变化

```undefined
后台返回：221629798350123011
前端接收：221629798350123000
```

最终前端接收的时候，这个值不是一个精确的值。大致了解了下，原因大致如下

JavaScript中表示数值的只有一个类型，就是Number，Number类型采用IEEE754标准来表示数字，不区分整数和浮点数，统一使用64bit 浮点数来进行表示。而在一段范围之内，所有的整数都有唯一的浮点数表示，这些整数叫做安全整数，而在JS里，安全整数的范围是：

```jsx
Number.MAX_SAFE_INTEGER  =   2^53 - 1        => 9007199254740991
Number.MIN_SAFE_INTEGER   = - (2^53 - 1)     => -9007199254740991
```



解决方法一：



解决方式也很简单了，就是返回的时候不用数值类型，使用字符串的形式进行返回。而在后台的处理中，有多种方式方式将Long类型转为字符串类型，这里我们了解两种常用的方式。

SpringBoot中默认使用的json解析工具是Jackson，使用的默认Message转换器是Jackson的`MappingJackson2HttpMessageConverter`，实际实现是通过该对象的ObjectMapper来实现的，所以我们可以修改一下ObjectMapper的一些设置：

```java
@Override
public void configureMessageConverters(List<HttpMessageConverter<?>> converters) {
    MappingJackson2HttpMessageConverter jackson2HttpMessageConverter =
            new MappingJackson2HttpMessageConverter();
    ObjectMapper objectMapper = new ObjectMapper();
    SimpleModule simpleModule = new SimpleModule();
    simpleModule.addSerializer(Long.class, ToStringSerializer.instance);
    objectMapper.registerModule(simpleModule);
    jackson2HttpMessageConverter.setObjectMapper(objectMapper);
    converters.add(jackson2HttpMessageConverter);
    converters.add(new StringHttpMessageConverter(Charset.forName("UTF-8")));
}
```

解决方法二：

针对特定类中的某个字段，那么这时候我们可以在实体中，通过配置Jackson的`JsonInclude`和`JsonSerialize`来实现：

```java
@JsonInclude(value = JsonInclude.Include.NON_EMPTY)
public class User implements Serializable{
    private String name;
    private String id;
    private Double aDouble;
    @JsonSerialize(using = ToStringSerializer.class)
    private Long sum;
    
    // set,get省略
}
```



解决方法三：

我们工程中经常会用到fastjson，如果我们想通过使用fastjson来实现该类型转换的话，可以通过如下配置：

```java
Override
public void configureMessageConverters(List<HttpMessageConverter<?>> converters) {
    MappingJackson2HttpMessageConverter jackson2HttpMessageConverter = new MappingJackson2HttpMessageConverter();

    ObjectMapper objectMapper = new ObjectMapper();
    /**
     * 序列换成json时,将所有的long变成string
     * 因为js中得数字类型不能包含所有的java long值
     */
    SimpleModule simpleModule = new SimpleModule();
    simpleModule.addSerializer(Long.class, ToStringSerializer.instance);
    simpleModule.addSerializer(Long.TYPE, ToStringSerializer.instance);
    objectMapper.registerModule(simpleModule);

    jackson2HttpMessageConverter.setObjectMapper(objectMapper);
    converters.add(jackson2HttpMessageConverter);
}
```

解决方法三：

使用ToStringSerializer的注解，让系统序列化时，保留相关精度

```java
@JsonSerialize(using=ToStringSerializer.class)
private Long createdBy;
```



博客地址：<https://www.codepeople.cn>

=====================================================================

微信公众号：

![](https://www.codepeople.cn/imges/weixin_icon/weixin.jpg)