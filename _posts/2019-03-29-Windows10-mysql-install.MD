---
title:  "Windows10安装MySQL绿色版方法"
categories: Windows10 MySQL
tags: Windows10 MySQL
author: LIUREN
---

# Windows10安装MySQL绿色版方法

> Windows10安装MySQL绿色版方法



## 安装环境

```shell
开发工具：STS
Maven版本：apache-maven-3.5.2
java jdk 1.8
MySQL版本：5.7
系统：Windows10
```



## 一.下载MySQL绿色安装包

<http://dev.mysql.com/downloads/mysql/5.7.html#downloads>

我们可以选择自己所需要的版本、环境、平台：

![](https://www.codepeople.cn/imges/MySQL/003.png)

## 二.安装

1）下载后，我们开始解压下载的压缩包

![](https://www.codepeople.cn/imges/MySQL/004.png)

2)配置my.ini

打开解压后的文件夹，我们看到： （my.ini是本人添加进去的）

![](https://www.codepeople.cn/imges/MySQL/005.png)

我们新建一个my.ini文件，内容如下就可以了

```ini
[client]
no-beep

# pipe
socket=0.0
port=3382

[mysql]
# 设置mysql客户端默认字符集
default-character-set=utf8mb4
[mysqld]
# The next three options are mutually exclusive to SERVER_PORT below.
# skip-networking

enable-named-pipe

shared-memory

shared-memory-base-name=MYSQL-3382

# The Pipe the MySQL Server will use
socket=MYSQL-3382
#设置端口为3382默认端口是3306
port=3382

# 设置mysql的安装目录
basedir=D:\mysql\3382
# 设置mysql数据库的数据的存放目录
datadir=D:\mysql\3382\data
# 允许最大连接数
max_connections=200
# 服务端使用的字符集默认为8比特编码的latin1字符集
character-set-server=utf8mb4
collation-server=utf8mb4_unicode_ci
# 创建新表时将使用的默认存储引擎
default-storage-engine=INNODB
```

注意一下设置自己的路径，basedir、datadir。文件编码为ANSI。

注意端口号和字节编码



## 三.命令行安装

首先配置环境变量

![](https://www.codepeople.cn/imges/MySQL/006.png)

![](https://www.codepeople.cn/imges/MySQL/007.png)

![](https://www.codepeople.cn/imges/MySQL/008.png)

***打开命令行请切换到mysql下的bin路径，否则会发生莫名错误。无论你是否设置了环境变量***

***用管理员身份打开命令行，然后输入 mysqld install 来安装mysqld        （mysqld remove是删除命令）***

![](https://www.codepeople.cn/imges/MySQL/010.png)

![](https://www.codepeople.cn/imges/MySQL/013.png)

命令如下：`MySQL-3382`是服务名称

```powershell
mysqld install MySQL-3382
```

再输入：mysqld --initialize      此时应该没有任何提示

```powershell
mysqld --initialize
```

如果报错，可以先删除data目录下的所有文件，然后再次执行

![](https://www.codepeople.cn/imges/MySQL/011.png)

启动MySQL服务

```
net start MySQL-3382
```



另注：第一次登陆MYSQL时，会提示要求输入初始密码，这是考虑安全因素，命令：mysqld --initialize会随机生成密码。

初始密码在上图data文件夹下的xxx.err文件中，可以用记事本打开找到

A temporary password is generated for root@localhost（可以用ctrl+f 查找功能）

其中 root@localhost后面的一串就即为初始密码

输入密码进入后可以用如下命令修改，这里密码改为root（当然可以改为任意你想要的）：

```
ALTER USER 'root'@'localhost' IDENTIFIED BY 'root';
```

![](https://www.codepeople.cn/imges/MySQL/012.png)

**MySQL绿色版安装完毕**



**MySQL5.6安装的时候默认密码为空，不需要密码直接就可以登录。**

```shell
#执行mysql初始化命令
mysqld --initialize-insecure --user=mysql
#设置root的密码为kelan12345!@#$%
set password for root@localhost = password('xxx13145');
#赋予别的用户可以远程连接mysql数据库
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%'IDENTIFIED BY 'xxx13145' WITH GRANT OPTION; 
#刷新权限【必须加上】
FLUSH PRIVILEGES;
```



=====================================================================

[博客地址](https://www.codepeople.cn)：<https://www.codepeople.cn>

=====================================================================

微信公众号：

![](https://www.codepeople.cn/imges/weixin_icon/weixin.jpg)