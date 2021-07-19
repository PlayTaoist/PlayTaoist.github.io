---
title:  "Windows中多个JDK和多个Tomcat问题,Tomcat指定JDK问题"
categories: Windows JDK Tomcat
tags: Windows JDK Tomcat
author: LIUREN
---

# Windows中多个JDK和多个Tomcat问题,Tomcat指定JDK问题

> 项目部署中我们经常碰到公司因为缺少服务器，不同的服务和开发程序安装到了同一台电脑上，早期服务可能使用的JDK和现在使用的JDK版本不一致，比如早期使用的是JDK1.7现在使用的是JDK1.8那么怎么让Tomcat可以使用不同的JDK启动服务呢？下面详细介绍使用方法
>
> 协议：CC BY-SA 4.0 https://creativecommons.org/licenses/by-sa/4.0/  
>
> 版权声明：本文为原创文章，遵循 CC 4.0 BY-SA 版权协议，转载请附上原文出处链接及本声明。



## 一、配置环境变量（配置其中一个就行）

为啥不用JDK1.8以上的，不要问我，问Oracle，他不免费没办法。下载地址和方法自行百度，你能搜到这个文章，找这两个JDK肯定没问题。安装完毕后配置**系统环境变量**

系统变量→新建 JAVA_HOME 变量 。

变量值填写jdk的安装目录（本人是 E:\Java\jdk1.7.0)

系统变量→寻找 Path 变量→编辑

在变量值最后输入 %JAVA_HOME%\bin;%JAVA_HOME%\jre\bin;

（注意原来Path的变量值末尾有没有;号，如果没有，先输入；号再输入上面的代码）

系统变量→新建 CLASSPATH 变量

变量值填写 .;%JAVA_HOME%\lib;%JAVA_HOME%\lib\tools.jar（注意最前面有一点）

系统变量配置完毕

只需要配置一个就行了，你想配置1.7或者1.8都可以

------



## 二、解决不同的tomcat使用不同的JDK问题。

前提条件是你已经安装好了两个JDK，JDK安装目录最好是自定义的。

假如：有tomcat8和tomcat7两个服务。

那么tomcat7可以直接使用系统的默认环境变量（端口改不改随你）

那么怎么让tomcat8使用JDK1.8呢？很简单

找到tomcat8/bin下的catalina.bat文件(Linux系统是catalina.sh)

然后在catalina.bat(Linux系统是catalina.sh)文件上加上配置文件：

```
set JAVA_HOME=D:\Java\jdk1.8.0_112
set JRE_HOME=D:\Java\jre1.8.0_112
```

Linux上在catalina.sh最上面加上如下配置

```
export JAVA_HOME=/home/Java/jdk1.8.0_112
export JRE_HOME=/home/Java/jre1.8.0_112
```

保存，然后启动start.bat(Linux上是启动start.sh) 就可以了。然后直接访问tomcat的主界面，就能看到不懂版本的tomcat 猫了。



=====================================================================

微信公众号：

![](https://www.codepeople.cn/imges/weixin_icon/weixin.jpg)