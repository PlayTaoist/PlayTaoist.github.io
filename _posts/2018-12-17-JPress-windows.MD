---
title:  "JPress的CMS系统在Window下的部署和使用"
categories: JPress
tags: JPress
author: LIUREN
---

# JPress的CMS系统在Window下的部署和使用

> JPress的CMS系统在Windows下的部署和使用。现在比较流行的是使用WordPress系统，因为系统比较成熟稳定，主要是使用广泛文档丰富，如果使用JPress不习惯，可以单独搭建WordPress。

 

## 开始项目



开始使用JPress系统的话首先要进入官网对其进行熟悉

**官网网址如下**：<http://www.jpress.io/faq.html>

然后是下载项目，项目下载地址是。<https://gitee.com/fuhai/jpress>

下载完项目后导入到Eclipse中，记得要安装Maven3.0版本以上的。

 

导入的效果如下：![001](http://www.codepeople.cn/imges/jpress/001.png)

 

**然后执行使用官网给的方法tomcat7:run-war 这个方法怎么也执行不了，添加上tomcat插件后还是不行**

```xml
<!-- 配置Tomcat插件 -->
            <plugin>
                <groupId>org.apache.tomcat.maven</groupId>
                <artifactId>tomcat7-maven-plugin</artifactId>
                <configuration>
                    <port>8081</port>
                    <path>/</path>
                </configuration>
            </plugin>
```

一直报找不到依赖报，根据报错信息对模块包进行maven install 后再次启动还是不行，最后放弃了。

后期抽时间再研究吧。没办法 想看网站的话，那就使用官网给的另外一种方法。

cmd命令行进入到jpress的文件夹目录下，执行 **mvn package**

执行成功了，然后在**jpress\starter-tomcat\target** 目录下找到打包好的**starter-tomcat-1.0.war**项目，把项目放到web服务器中，启动服务。

我是直接放到tomcat目录下，然后启动。

记得配置数据库

配置成功后，启动tomcat（运行 `tomcat/bin/startup.bat`），浏览器输入 `http://127.0.0.1:8080/starter-tomcat-1.0` 即可访问。若把 `tomcat/webapp/starter-tomcat-1.0` 里面的文件拷贝到 `tomcat/webapp/ROOT`，访问`http://127.0.0.1:8080`即可。

然后进入数据库配置页面，配置完毕可以进入后台了。

http://127.0.0.1:8080/starter-tomcat-1.0/admin/login   是后台登陆界面

http://127.0.0.1:8080/starter-tomcat-1.0/     是生成的前台模板页面

![001](http://www.codepeople.cn/imges/jpress/002.png)

![001](http://www.codepeople.cn/imges/jpress/003.png)