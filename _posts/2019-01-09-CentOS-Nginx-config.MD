---
title:  "Nginx配置https的部署实践"
categories: Nginx
tags: Nginx
author: LIUREN
---



# Nginx配置https的部署实践

> http以明文的形式在浏览器和服务器之间交换数据，没有任何数据加密，攻击者可以在截取之间的信息并读懂，这明显不安全，所以现在浏览器浏览器都要求网站域名配置SSL域名证书，以[https](https://www.centos.bz/tag/https/)协议传输内容。
>



那问题来了：

![003](http://www.codepeople.cn/imges/centos/nginx/003.jpg)



## HTTP与HTTPS

- HTTP：超文本传输协议
- HTTPS：超文本传输安全协议

简单来说，可以用这个公式：HTTPS = HTTP + SSL

- SSL：安全套接层，一种安全协议

也就是说：

为了数据传输的安全，HTTPS在HTTP的基础上加入了SSL协议，SSL依靠证书来验证服务器的身份，并为浏览器和服务器之间的通信加密。

![004](http://www.codepeople.cn/imges/centos/nginx/004.jpg)

![005](http://www.codepeople.cn/imges/centos/nginx/005.jpg)

## 申请ssl域名证书

登录腾讯/阿里云之后，找到SSL证书管理栏目，购买或者申请域名证书。

注意：

- 免费证书有效期是一年，一年后手动重新申请即可
- 域名需要备案

![006](http://www.codepeople.cn/imges/centos/nginx/006.jpg)



申请完了之后，就可以下载证书啦~



![006](http://www.codepeople.cn/imges/centos/nginx/007.png)



如图下载的域名证书，可以配置到[Apache](https://www.centos.bz/category/web-server/apache/)、[Nginx](https://www.centos.bz/category/web-server/nginx/)、[Tomcat](https://www.centos.bz/category/web-server/tomcat/)等服务器上面。



![006](http://www.codepeople.cn/imges/centos/nginx/008.gif)



## nginx配置https步骤

好，接下来我们进入正题，给[nginx](https://www.centos.bz/tag/nginx-2/)配置域名证书嘿~

解压下载下来的域名证书，获取Nginx里面的两个文件。

![006](http://www.codepeople.cn/imges/centos/nginx/009.png)

- crt文件是以PEM格式存在的证书，可以用于不同的程序和设备

- key文件是授权文件

  

  ## 第一步：

  把crt和key文件上传到nginx的conf目录下。

![006](http://www.codepeople.cn/imges/centos/nginx/010.jpg)



## 第二步：

nginx.conf或自定义配置文件上配置SSL证书。

![006](http://www.codepeople.cn/imges/centos/nginx/011.jpg)



**HTTPS的默认端口是443，就像HTTP的默认端口80一样，从图中可以看到，这个服务最后代理的是8080端口的[tomcat](https://www.centos.bz/tag/tomcat/)。**



## 第三步：

配置完了第二步已经完成一大半了，只要用户输入https://www.java-mindmap.com就可以访问我的社区网站，但是一般用户都懒得输入https://，而不输入的话默认就是发起http链接，所以，需要还需要配置http强制转换成https的链接。

![006](http://www.codepeople.cn/imges/centos/nginx/012.png)



这样，当用户访问http链接时候，强制转成了https的服务了。



![006](http://www.codepeople.cn/imges/centos/nginx/013.gif)





至此，HTTPS配置成功~



![006](http://www.codepeople.cn/imges/centos/nginx/014.jpg)







## 来源：

文章原文来源：<https://www.itcodemonkey.com/article/4674.html>

