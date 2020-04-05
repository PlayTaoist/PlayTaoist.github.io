---
title:  "SpringBoot2.x打包方法浅谈"
categories: SpringBoot2.0
tags: SpringBoot2.0
author: LIUREN
---

# SpringBoot2.x打包方法浅谈

> 帮朋友做了几个小功能，使用的是renren的框架地址如下：<https://gitee.com/renrenio/renren-security> 但是在打包的时候出现问题，我到底是打成`jar`、`war`包，最后决定都打，jar还好说，war的时候一不小心掉坑里了，虽然爬出来了，但是想留个记录。



## 开发框架

```shell
框架：SpringBoot2.0
java jdk 1.8
开发工具：Eclipse
数据库：MySQL
```



## 一.首先打成jar包

renren网的架构默认是jar包：

直接 右键renren-admin项目=> Run As =>  maven install 然后会自动生成jar包，可以在`renren-admin\target`的目录下下看到`renren-admin.jar`文件，如果需要修改数据库配置文件，可以直接使用压缩文件的打开方式打开，然后修改`application-dev.yml`中的数据库配置信息

运行jar文件

运行文件首要条件：1.数据库已经配置完毕；2.JDK环境已经配置完毕

运行方式：

`java -jar renren-admin.jar `

访问地址：http://localhost:8080/renren-admin



## 二.打包成war包

修改renren-admin的pom.xml文件把`<packaging>jar</packaging>`修改成`<packaging>war</packaging>`

然后修改renren-admin的主启动文件`AdminApplication.java`

修改成如下：

```java
/**
 * Copyright (c) 2016-2019 人人开源 All rights reserved.
 *
 * https://www.renren.io
 *
 * 版权所有，侵权必究！
 */

package io.renren;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.builder.SpringApplicationBuilder;
import org.springframework.boot.web.servlet.support.SpringBootServletInitializer;


@SpringBootApplication
public class AdminApplication extends SpringBootServletInitializer {

    public static void main(String[] args) {
        SpringApplication.run(AdminApplication.class, args);
    }

    // 不重写打包war部署到tomcat接口会报404
    @Override
    protected SpringApplicationBuilder configure(SpringApplicationBuilder application) {
        return application.sources(AdminApplication.class);
    }

}
```

注意：记得使其继承`SpringBootServletInitializer` 然后添加main方法，再次打包，打包方式和jar一样，然后会在`renren-admin\target`目录下生成`renren-admin.war`包，把该包放到tomcat的webapps目录下，记得线删除webapps目录下的所有文件，然后再copy进去【记得修改数据库配置信息】。运行tomcat，然后就可以直接访问了：

访问地址：http://localhost:8080/renren-admin

如果不修改`AdminApplication.java`文件，虽然能打包成功，但是启动后不能访问直接报404



## 三.打包成Docker镜像文件

这个可以自行百度，需要做配置信息，正在学习中，在这里暂时不讲解。



=======================================================================================

博客地址：<https://www.codepeople.cn>

=======================================================================================

微信公众号：

![](https://www.codepeople.cn/imges/weixin_icon/weixin.jpg)