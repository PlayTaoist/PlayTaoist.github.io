---
title:  "CentOS7安装Zabbix5.0+MySQL+Nginx（亲测成功）"
categories:  Zabbix5.0 CentOS7 Nginx MySQL
tags: Zabbix5.0 CentOS7 Nginx MySQL
author: LIUREN
---

# CentOS7安装Zabbix5.0+MySQL+Nginx（亲测成功）

> CentOS7安装Zabbix5.0+MySQL+Nginx（亲测成功）
>
> 协议：CC BY-SA 4.0 https://creativecommons.org/licenses/by-sa/4.0/  
>
> 版权声明：本文为原创文章，遵循 CC 4.0 BY-SA 版权协议，转载请附上原文出处链接及本声明。



[TOC]

**服务器操作系统版本：**

```shell
[root@localhost ~]# cat /etc/redhat-release
CentOS Linux release 7.9.2009 (Core)
```

**关闭防火墙**

```shell
systemctl stop firewalld
```

**开机禁用防火墙自启**

```shell
systemctl disable firewalld
```

**设置临时关闭selinux**

```shell
setenforce 0
```

**设置永久关闭selinux**

```shell
vim /etc/sysconfig/selinux 
```

将第7行内容修改如下

```shell
SELINUX=disabled
```

修改完成之后使用 `reboot` 命令重启服务器即可开始zabbix的安装

**配置yum源**

```shell
rpm -Uvh https://repo.zabbix.com/zabbix/5.0/rhel/7/x86_64/zabbix-release-5.0-1.el7.noarch.rpm
yum clean all
```

**安装服务端以及客户端**

```shell
yum install zabbix-server-mysql zabbix-agent  zabbix-get
```

**安装zabbix前端**

```shell
yum install centos-release-scl
```

**编辑配置文件 /etc/yum.repos.d/zabbix.repo**

```shell
vim  /etc/yum.repos.d/zabbix.repo
[zabbix-frontend]
...
enabled=1
```

**安装Zabbix和mysql连接工具已经nginx配置文件**

```shell
yum install zabbix-web-mysql-scl zabbix-nginx-conf-scl
```

**安装mariadb启动**

```shell
yum -y install mariadb mariadb-devel mariadb-server 
```

**启动数据库**

```shell
systemctl start mariadb 
```

**设置开机启动**

```shell
systemctl enable mariadb
```

**创建初始数据库**(一般默认密码为空) 

```shell
mysql -uroot -p
password
mysql> create database zabbix character set utf8 collate utf8_bin;
mysql> create user zabbix@localhost identified by 'password';
mysql> grant all privileges on zabbix.* to zabbix@localhost;
mysql> quit;
```

**导入zabbix数据库**

```shell
zcat /usr/share/doc/zabbix-server-mysql*/create.sql.gz | mysql -uzabbix -p zabbix
```

**修改zabbix-server的配置文件**

```shell
vim /etc/zabbix/zabbix_server.conf
#数据库服务器ip
DBHost=127.0.0.1 
#数据库库名
DBName=zabbix
#连接数据库用户
DBUser=zabbix
#密码
DBPassword=password
#端口
DBPort=3306
#默认是ServerActive=127.0.0.1 增加一个监控IP
StatsAllowedIP=127.0.0.1,192.168.0.180
```

**修改zabbix_agend.conf的配置文件**

```shell
vim /etc/zabbix/zabbix_agentd.conf
#主动监控的IP默认是ServerActive=127.0.0.1 192.168.180是自行添加的自己机器的IP
ServerActive=127.0.0.1,192.168.0.180
```

**修改nginx配置文件**

```shell
vim /etc/opt/rh/rh-nginx116/nginx/conf.d/zabbix.conf
```

**将2、3行内容修改如下**
server_name 为所在主机的IP地址

```shell
listen          80;
server_name     192.168.0.180;
```

将32行内容注释掉，并新增33行内容如下

```shell
# fastcgi_pass    unix:/var/opt/rh/rh-php73/run/php-fpm/zabbix.sock;
fastcgi_pass   127.0.0.1:9000;
```

**修改PHP配置文件**

```shell
vim /etc/opt/rh/rh-php72/php.ini
```

将383行修改如下

```shell
max_execution_time = 300
```

将393行修改如下

```shell
max_input_time = 300
```

将672行修改如下

```shell
post_max_size = 16M
```

将902行修改如下

```shell
date.timezone = Asia/Shanghai
```

**重启所有服务并设为开机自启**

```shell
systemctl restart zabbix-server zabbix-agent rh-nginx116-nginx rh-php72-php-fpm
```

```shell
systemctl enable zabbix-server zabbix-agent rh-nginx116-nginx rh-php72-php-fpm
```

**浏览器输入 `IP/index.php` 配置zabbix**

![](https://www.codepeople.cn/imges/00x130.png)

![](https://www.codepeople.cn/imges/00x131.png)

**连数据库的密码为 4.3 中创建zabbix用户时设定的密码，本实验为：password**

![](https://www.codepeople.cn/imges/00x132.png)

![](https://www.codepeople.cn/imges/00x133.png)

![](https://www.codepeople.cn/imges/00x134.png)

![](https://www.codepeople.cn/imges/00x135.png)

**用户名默认为 `Admin`，密码默认为 `zabbix`**

![](https://www.codepeople.cn/imges/00x136.png)

**成功访问zabbix界面**

![](https://www.codepeople.cn/imges/00x137.png)

**汉化：**

![](https://www.codepeople.cn/imges/00x138.png)

![](https://www.codepeople.cn/imges/00x139.png)

![](https://www.codepeople.cn/imges/00x140.png)

**解决中文乱码问题**

1)在zabbix-server服务器上安装如下文件符集

```shell
yum -y install wqy-microhei-fonts
```

2)然后替换linux上默认的字符集

```shell
cp /usr/share/fonts/wqy-microhei/wqy-microhei.ttc /usr/share/fonts/dejavu/DejaVuSans.ttf
```

3)出现覆盖询问时输入 `y` 回车即可

```shell
[root@host-137 ~]# cp /usr/share/fonts/wqy-microhei/wqy-microhei.ttc /usr/share/fonts/dejavu/DejaVuSans.ttf
cp: overwrite ‘/usr/share/fonts/dejavu/DejaVuSans.ttf’? y
[root@host-137 ~]# 
```

4)查看效果，乱码问题成功解决

![](https://www.codepeople.cn/imges/00x141.png)

=====================================================================

微信公众号：

![](https://www.codepeople.cn/imges/weixin_icon/weixin.jpg)