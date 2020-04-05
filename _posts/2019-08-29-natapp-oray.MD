---
title:  "内网穿透工具简介"
categories: windows
tags: windows
author: LIUREN
---

# 内网穿透工具简介

> 很多朋友做开发的时候需要使用到域名访问自己的项目；没有独立的主机IP，也不想买。但有时候就是需要提供外网访问功能。甚至部署到自己部门的服务器上但苦于没有域名；IP有不能从外网访问。现在给大家介绍两款常用的产品方便需要的时候可以直接拿来做参考。



###  一、开启您的内网穿透之旅

NATAPP ： <https://natapp.cn/tunnel/buy>

用户可以直接去免费注册，然后根据系统下载所需的版本；

我直接下载了Windows版本。现在给大家介绍一下Windows版本怎么开启和使用

1.打开https://natapp.cn/tunnel/buy网址，然后注册

2.登录网站 如图

![](https://www.codepeople.cn/imges/windows/001.png)

3.然后弹出如下界面

![](https://www.codepeople.cn/imges/windows/002.png)

![](https://www.codepeople.cn/imges/windows/003.png)

4.确定，然后复制编码

![](https://www.codepeople.cn/imges/windows/004.png)

5.客户端下载

下载地址：<https://natapp.cn/#download>

下载完毕后直接解压，**记得目录中最好不要有中文**

如图

![](https://www.codepeople.cn/imges/windows/005.png)

6.启动

![](https://www.codepeople.cn/imges/windows/006.png)

![](https://www.codepeople.cn/imges/windows/007.png)

执行的命令如下：8fcc46357c5d308d 这个是刚刚从网页上copy的值

```shell
natapp -authtoken=8fcc46357c5d308d
```

执行成功后如图

![](https://www.codepeople.cn/imges/windows/008.png)

然后就能通过域名访问你的系统了。HTTPS的要收费的，不做介绍了。



Oray花生壳搭建相对来说比较熟悉

1.登录注册，具体网址百度

2.然后免费注册一个域名

3.然后配置做内网穿透

花生壳不太稳定。但是他的域名是固定的，这一点比较好。重启服务后域名也不会改变



如何取舍根据跟人情况去选择



=======================================================================================

博客地址：<https://www.codepeople.cn>

=======================================================================================

微信公众号：

![](https://www.codepeople.cn/imges/weixin_icon/weixin.jpg)