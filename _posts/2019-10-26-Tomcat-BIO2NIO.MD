---
title:  "Eclipse中开发环境也想把Tomcat 的默认BIO模式改为NIO模式"
categories: Tomcat
tags: Tomcat
author: LIUREN
---

# Eclipse中开发环境也想把Tomcat 的默认BIO模式改为NIO模式

> 有时候，开发环境我们也想把Tomcat 的默认BIO模式改为NIO模式，该如何改呢？



#### 一、 问题

有时候，开发环境我们也想把Tomcat 的默认BIO模式改为NIO模式，该如何改呢？。

#### 二、方案

通过eclipse里面的server.xml进行修改。

#### 三、步骤

首先我们来一起看看eclipse默认的server.xml文件

名词解释：

bio(blocking I/O)，顾名思义，即阻塞式I/O操作，表示Tomcat使用的是传统的Java I/O操作(即java.io包及其子包)。Tomcat在默认情况下，就是以bio模式运行的。遗憾的是，就一般而言，bio模式性能比较地下。我们可以通过Tomcat Manager来查看服务器的当前状态。

其中ajp端口是用于和服务器负载均衡时使用的。

nio(new I/O)，是Java SE 1.4及后续版本提供的一种新的I/O操作方式(即java.nio包及其子包)。Java nio是一个基于缓冲区、并能提供非阻塞I/O操作的Java API，因此nio也被看成是non-blocking I/O的缩写。

它拥有比传统I/O操作(bio)更好的并发运行性能。要让Tomcat以nio模式来运行也比较简单，我们只需要在Tomcat安装目录/conf/server.xml文件中将如下配置：

```xml
<Connector connectionTimeout="20000" port="8080" protocol="HTTP/1.1" redirectPort="8443"/>
```

改成

```xml
<Connector connectionTimeout="20000" port="8080" protocol="org.apache.coyote.http11.Http11NioProtocol" redirectPort="8443"/>
```

如果需要修改ajp，可以这样修改，只需要把

```xml
<Connector port="8009" protocol="AJP/1.3" redirectPort="8443"/>
```

改成

```xml
<Connector port="8009" protocol="org.apache.coyote.ajp.AjpNioProtocol" redirectPort="8443" />
```



原文链接：<http://www.souvc.com/?p=2850>

=====================================================================

博客地址：<https://www.codepeople.cn>

=====================================================================

微信公众号：

![](https://www.codepeople.cn/imges/weixin_icon/weixin.jpg)