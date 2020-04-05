---
title:  "Nginx+Tomcat实现静态和动态页面的分离"
categories: Tomcat
tags: Tomcat
author: LIUREN
---

# Nginx+Tomcat实现静态和动态页面的分离

> Centos环境下使用Nginx+Tomcat实现静态和动态页面的分离



#### 一、 问题

在配置文件中，静态HTML网页，图片，JS，CSS、Frash等使用Nginx来处理，以便获得更快的速度，文件拓展名为.jsp、.do的请求，由Nginx反向代理TomcatHttp服务器来处理。

#### 二、方案

公司的项目之前一直都是使用Tomcat处理用户请求的，无论静态还是动态的东西，一律交给Tomcat处理。Tomcat主要是负责处理Servlet的，静态的文件还是交给Nginx处理，Nginx对静态文件的处理比Tomcat不是只快了一点，并且Nginx的使用对项目并发能力有很大的提升。

#### Nginx的安装

Nginx的安装过程可以参考：[CentOS 7 下 Nginx安装以及配置](http://www.souvc.com/?p=1661)，以下主要是配置Nginx实现静态和动态页面的分离。

#### Nginx的配置

查看Nginx配置文件

```shell
cd /usr/local/nginx/conf
cat nginx.conf
vi nginx.conf
```

下面主要记录下主要的配置过程，简单的配置文件如下：

主要修改端口号，以及服务器的server_name名字，资源文件放置的位置。

```nginx
server {
        listen       8866;
        server_name  localhost;
        location / {
            root   /opt/ftp/souvc/;
            index  index.html index.htm;
        }
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
    }
```

以下是详细的配置：

```nginx
#user  nobody;
worker_processes  1;
 
#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;
 
#pid        logs/nginx.pid;
 
 
events {
    worker_connections  1024;
}
 
 
http {
    include       mime.types;
    default_type  application/octet-stream;
 
    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';
 
    #access_log  logs/access.log  main;
 
    sendfile        on;
    #tcp_nopush     on;
 
    #keepalive_timeout  0;
    keepalive_timeout  65;
 
    #gzip  on;
 
    server {
        listen       8866;
        server_name  localhost;
        location / {
            root   /opt/ftp/souvc/;
            index  index.html index.htm;
        }
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
    }
 
 
    # another virtual host using mix of IP-, name-, and port-based configuration
    #
    #server {
    #    listen       8000;
    #    listen       somename:8080;
    #    server_name  somename  alias  another.alias;
 
    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}
 
 
    # HTTPS server
    #
    #server {
    #    listen       443 ssl;
    #    server_name  localhost;
 
    #    ssl_certificate      cert.pem;
    #    ssl_certificate_key  cert.key;
 
    #    ssl_session_cache    shared:SSL:1m;
    #    ssl_session_timeout  5m;
 
    #    ssl_ciphers  HIGH:!aNULL:!MD5;
    #    ssl_prefer_server_ciphers  on;
 
    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}
 
}
```

#### 效果测试

上传一张图片到 /opt/fto/souvc/下，我的图片名字为：Koala.jpg

访问一下，配置Nginx的服务器ip：http://192.168.1.27:8866/Koala.jpg

效果如下

原文链接：<http://www.souvc.com/?p=2488>

=====================================================================

博客地址：<https://www.codepeople.cn>

=====================================================================

微信公众号：

![](https://www.codepeople.cn/imges/weixin_icon/weixin.jpg)