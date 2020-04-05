---
title:  "Fiddler使用方法讲解"
categories: Fiddler
tags: Fiddler
author: LIUREN
---

# Fiddler使用方法讲解

> Fiddler是一款抓包工具，主要可以进行抓包分析和测试。下面讲述使用方法



## 安装环境

```shell
安装环境：Windows 10
```



## 一.下载Fiddler安装包

下载[Fiddler](https://www.telerik.com/download/fiddler)地址：<https://www.telerik.com/download/fiddler>

下载结果如下：

![](https://www.codepeople.cn/imges/fiddler/001.png)

下载完成后直接选择默认安装，不需要手动设置

进入安装目录，找到`Fiddler.exe`双击启动`Fiddler`程序

![](https://www.codepeople.cn/imges/fiddler/002.png)

启动成功后的界面如下：

![](https://www.codepeople.cn/imges/fiddler/003.png)

## 二.设置Fiddler

Tools-> Fiddler Options (**配置完后记得要重启Fiddler**) 选中"**Decrpt HTTPS traffic**", Fiddler就可以截获HTTPS请求 选中"**Allow remote computers to connect**". 是允许别的机器把HTTP/HTTPS请求发送到Fiddler上来

![](https://www.codepeople.cn/imges/fiddler/004.png)

![](https://www.codepeople.cn/imges/fiddler/005.png)

## 三.打开浏览器开始抓包

为了测试效果，我们选择FireFox浏览器，因为Firefox浏览器可以清晰看到请求数据

​	注意看请求的地址信息和登录的账号密码

![](https://www.codepeople.cn/imges/fiddler/006.png)

​	Fiddler抓取的信息如下：可以看到登录的账号和密码是:admin/12356，登录成功后会跳转到成功页面所以http的状态码是 `302` 重定向

![](https://www.codepeople.cn/imges/fiddler/007.png)

Google浏览器访问地址抓取信息：因为Google做了数据交互，所以会不间断的发送请求，影响调试，不喜欢可以换成IE也行

![](https://www.codepeople.cn/imges/fiddler/008.png)

![](https://www.codepeople.cn/imges/fiddler/009.png)

## 四.抓取手机请求信息报

首先需要保证手机和电脑处在同一个局域网环境中

然后设置手机wifi的代理设置：例如：IPhone SE的代理设置如下

![](https://www.codepeople.cn/imges/fiddler/010.png)

![](https://www.codepeople.cn/imges/fiddler/011.png)

![](https://www.codepeople.cn/imges/fiddler/012.png)

开始使用手机发送请求

![](https://www.codepeople.cn/imges/fiddler/013.png)

![](https://www.codepeople.cn/imges/fiddler/014.png)

Fiddler抓取手机发送的请求信息

![](https://www.codepeople.cn/imges/fiddler/015.png)

`Fiddler`的简单抓包就是如此，具体的一些详细信息可以参考如下[博客](https://www.cnblogs.com/yyhh/p/5140852.html)<https://www.cnblogs.com/yyhh/p/5140852.html>

=======================================================================================

博客地址：<https://www.codepeople.cn>

=======================================================================================

微信公众号：

![](https://www.codepeople.cn/imges/weixin_icon/weixin.jpg)