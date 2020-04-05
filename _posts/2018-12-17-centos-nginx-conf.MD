---
title:  "CentOS 7 安装Nginx"
categories: Nginx
tags: Nginx CentOS7
author: LIUREN
---

## CentOS 7 安装Nginx

> *Nginx* (engine x) 是一个高性能的[HTTP](https://baike.baidu.com/item/HTTP)和[反向代理](https://baike.baidu.com/item/%E5%8F%8D%E5%90%91%E4%BB%A3%E7%90%86/7793488)服务，也是一个IMAP/POP3/SMTP服务。Nginx是由伊戈尔·赛索耶夫为[俄罗斯](https://baike.baidu.com/item/%E4%BF%84%E7%BD%97%E6%96%AF/125568)访问量第二的Rambler.ru站点（俄文：Рамблер）开发的，第一个公开版本0.1.0发布于2004年10月4日。
>
> 其将[源代码](https://baike.baidu.com/item/%E6%BA%90%E4%BB%A3%E7%A0%81)以类BSD许可证的形式发布，因它的稳定性、丰富的功能集、示例配置文件和低系统资源的消耗而[闻名](https://baike.baidu.com/item/%E9%97%BB%E5%90%8D/2303308)。2011年6月1日，nginx 1.0.4发布。
>
> Nginx是一款[轻量级](https://baike.baidu.com/item/%E8%BD%BB%E9%87%8F%E7%BA%A7/10002835)的[Web](https://baike.baidu.com/item/Web/150564) 服务器/[反向代理](https://baike.baidu.com/item/%E5%8F%8D%E5%90%91%E4%BB%A3%E7%90%86/7793488)服务器及[电子邮件](https://baike.baidu.com/item/%E7%94%B5%E5%AD%90%E9%82%AE%E4%BB%B6/111106)（IMAP/POP3）代理服务器，并在一个BSD-like 协议下发行。其特点是占有内存少，[并发](https://baike.baidu.com/item/%E5%B9%B6%E5%8F%91/11024806)能力强，事实上nginx的并发能力确实在同类型的网页服务器中表现较好，中国大陆使用nginx网站用户有：百度、[京东](https://baike.baidu.com/item/%E4%BA%AC%E4%B8%9C/210931)、[新浪](https://baike.baidu.com/item/%E6%96%B0%E6%B5%AA/125692)、[网易](https://baike.baidu.com/item/%E7%BD%91%E6%98%93/185754)、[腾讯](https://baike.baidu.com/item/%E8%85%BE%E8%AE%AF/112204)、[淘宝](https://baike.baidu.com/item/%E6%B7%98%E5%AE%9D/145661)等。



## CentOS 7 安装Nginx

#### 1.安装准备

```shell
首先由于nginx的一些模块依赖一些lib库，所以在安装nginx之前，必须先安装这些lib库，这些依赖库主要有g++、gcc、openssl-devel、pcre-devel和zlib-devel 所以执行如下命令安装
# yum install gcc
# yum install g++
# yum install pcre pcre-devel
# yum install zlib zlib-devel
# yum install openssl openssl--devel
```

#### 2.安装Nginx

```shell
安装之前，最好检查一下是否已经安装有nginx
# find -name nginx  
如果系统已经安装了nginx，那么就先卸载
# yum remove nginx
```

#### 3.新建一个用户

```shell
需要新建一个用户，禁止使用root用户进行操作
# useradd  -d ucenter
# passwd ucenter
# cd /ucenter
# mkdir soft
# chown ucenter:ucenter /ucenter/ -R
```

#### 4.安装在线下载包

```shell
在线安装wget包
# yum -y install wget
```

#### 5.在soft目录下直接下载和安装nginx

```shell
$ cd /ucenter/soft
$ mkdir nginx
$ wget http://nginx.org/download/nginx-1.14.1.tar.gz
#解压nginx压缩包
$ tar -vxf nginx-1.14.1.tar.gz
#解压后会产生一个nginx-1.14.1的目录，进入这个目录
$ cd nginx-1.14.1
```

#### 6.切换到root目录，然后指定默认的安装目录

```shell
# ./configure --prefix=/ucenter/soft/nginx --user=ucenter --group=ucenter
```

#### 7.再次切换用户到ucenter,然后进入nginx-1.14.1目录下进行编译安装

```shell
$ make
$ make install
```

#### 8.安装完毕后，启动nginx

```shell
$ cd /ucenter/soft/nginx
$ cd sbin
$ ./nginx
```

#centos中启动，重新加载，停止命令如下

```shell
./nginx    -------开启nginx服务
./nginx -s reload ----------重新加载nginx服务
./nginx -s stop ----------关闭nginx服务
```

#### 9.启动后记得到浏览器中访问，如果访问不了，记得在防火墙开启的情况下打开80端口就可以了，打开80端口的方法如下：

```shell
# firewall-cmd --zone=public --add-port=80/tcp --permanent
# /sbin/iptables -I INPUT -p tcp --dport 3306 -j ACCEPT
# /etc/rc.d/init.d/iptables save
# /etc/init.d/iptables status
```



> ##### 要到问题需要注意：

```
centos7 nginx: [emerg] bind() to 0.0.0.0:80 failed (13: Permission denied)
```

如果切换到普通用户下启动，有可能会报这个错误，怎么解决了？

执行如下命令：

```shell
$ su root
$ passwd --输入密码
# cd /ucenter/soft/nginx/sbin/
# ll 
# chown root nginx
# chmod u+s nginx
# ll
# su ucenter
$ ./nginx
$ ps -ef|grep nginx
```

#### 10.如何结束掉nginx进程

```shell
#一般情况下直接可以直接使用nginx的自带命令
$ ./nginx -s stop
#如果命令不起作用则可以使用
$ ps -ef | grep nginx
root      13064      1  0 16:46 ?        00:00:00 nginx: master process ./nginx
ucenter   13065  13064  0 16:46 ?        00:00:00 nginx: worker process
ucenter   13069  12602  0 16:46 pts/2    00:00:00 grep --color=auto nginx
#然后分别杀死主进程和分进程
$ kill -9 13064
$ kill -9 13065
#或者可以通过如下命令直接停止
$ pkill -9 nginx
```

#### 11.设置nginx开机启动

①在系统服务目录里创建nginx.service文件

```shell
cat>>/usr/lib/systemd/system/nginx.service
```

②写入内容如下：

```shell
[Unit]
Description=nginx
After=network.target
  
[Service]
Type=forking
ExecStart=/usr/local/nginx/sbin/nginx
ExecReload=/usr/local/nginx/sbin/nginx -s reload
ExecStop=/usr/local/nginx/sbin/nginx -s quit
PrivateTmp=true
  
[Install]
WantedBy=multi-user.target
```

[Unit]: 服务的说明

 Description:描述服务
 After:描述服务类别
 [Service]服务运行参数的设置
 Type=forking是后台运行的形式
 ExecStart为服务的具体运行命令
 ExecReload为重启命令
 ExecStop为停止命令
 PrivateTmp=True表示给服务分配独立的临时空间
 注意：[Service]的启动、重启、停止命令全部要求使用绝对路径
 [Install]运行级别下服务安装的相关设置，可设置为多用户，即系统运行级别为3

③设置开机自启动

`systemctl enable nginx.service`

④查看nginx状态

`systemctl status nginx.service`

![](https://www.codepeople.cn/imges/xxx.png)

很奇怪，明明启动成功了，为什么显示Active: inactive (dead)？

⑤杀死nginx重启nginx

`pkill -9 nginx`

`ps aux | grep nginx`

`systemctl start nginx`

再次查看状态，变成了active，搞定。

⑥重启服务器

`reboot`

⑦再次连接后，查看服务状态

`systemctl status nginx.servic`

======================================END=============================================

微信公众号：

![](https://www.codepeople.cn/imges/weixin_icon/weixin.jpg)