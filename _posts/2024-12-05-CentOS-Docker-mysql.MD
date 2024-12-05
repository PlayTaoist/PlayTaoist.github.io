---
title:  "Docker中的mysql配置"
categories: Docker mysql
tags: Docker mysql
author: LIUREN
---

# Docker中的mysql的安装和配置

> Docker中的mysql的安装和配置

## Docker中的mysql的安装和配置

### 1.启动Docker

```shell
启动：sudo systemctl start docker
	 service docker start
停止：systemctl stop docker
重启：systemctl restart docker
```

### 2.查询mysql

```shell
docker search mysql
```

### 3.安装mysql

3.1默认拉去最新版本mysql

```shell
docker pull mysql
```

3.2拉取指定版本mysql_5.7

```shell
sudo docker pull mysql:5.7
```

### 4.查看镜像

```shell
docker images
```

### 5.本地创建挂载目录-v

5.1启动MySQL容器

指定版本

```shell
sudo docker run -p 3306:3306 --name mysql \
-v /mydata/mysql/log:/var/log/mysql \
-v /mydata/mysql/data:/var/lib/mysql \
-v /mydata/mysql/conf:/etc/mysql \
-e MYSQL_ROOT_PASSWORD=root \
-d mysql:5.7
```

最新版本

```shell
docker run -p 3306:3306 --name mysql \
-v /mydata/mysql/log:/var/log/mysql \
-v /mydata/mysql/data:/var/lib/mysql \
-v /mydata/mysql/conf:/etc/mysql \
-e MYSQL_ROOT_PASSWORD=root \
-d mysql:latest
```

命令含义

```shell
docker run:在docker中启动一个容器实例
-d:该容器在后台运行
-p 3306:3306：容器与主机映射端口为，主机3306，容器3306
--name mysql：容器运行后的名称
-v /mysqldata/mysql/log:/var/log/mysql：将容器/var/log/mysql目录下的数据，备份到主机的 /mysqldata/mysql/log目录下
-v /mysqldata/mysql/data:/var/lib/mysql：将容器/var/lib/mysql目录下的数据，备份到主机的 /mysqldata/mysql/data目录下
-v /mysqldata/mysql/conf:/etc/mysql：将容器/etc/mysql目录下的数据，备份到主机的 mysqldata/mysql/conf目录下
-e MYSQL_ROOT_PASSWORD=root：设置当前mysql实例的密码为root
mysql:5.7:需要运行的容器名称以及版本号
```

5.2在 var/lib/docker/containers/ 目录下查看容器

```shell
cd /var/lib/docker/containers/
```

5.3查看mysql进程获取name-mysql名称

```shell
docker ps -a
```

5.4进入MySQL容器，并登录MySQL

```shell
格式：docker exec -it   mysql名称   bash
进入mysql容器操作台命令：docker exec -it mysql bash
登录mysql命令：mysql -u root -p
输入密码：
```

5.5开启远程访问权限

```shell
命令：use mysql;
命令：select host,user from user;
命令：ALTER USER 'root'@'%' IDENTIFIED WITH mysql_native_password BY '123456';
命令：flush privileges;
把root用户的密码改成 mysql_native_password 模式，即可远程连接
#创建一个账号-admin，用来进行远程访问；
CREATE USER 'admin'@'%' IDENTIFIED BY '123456';
#赋予所有权限给之前创建的账号:admin
GRANT ALL ON *.* TO 'admin'@'%';
#确认使用密码{123456}登录此账号{admin}
#密码尽量复杂，安全性更高。
ALTER USER 'admin'@'%' IDENTIFIED WITH mysql_native_password BY '123456';
#刷新权限
FLUSH PRIVILEGES;
```

5.6查看docker日志

```shell
docker logs -f --tail 10 a4dac74d48f7
```

5.7常用命令

查看进程

```shell
docker ps -a
```

关闭docker中mysql容器

```sh
docker stop mysqlserver
```

5.8重启docker中的mysql

重启docker

```shell
systemctl restart docker
```

关闭docker

```shell
systemctl stop docker
```

自动重启容器

```shell
sudo docker update mysql --restart=always
```

5.9移除原来的container ID

```shell
docker rm + container ID
```

### 6.MySQL设置

6.1 进入容器

```shell
docker exec -it mysql bash
mysql -u root -p
```

6.2定位容器位置

```shell
whereis mysql
#显示结果
mysql: /usr/bin/mysql /usr/lib/mysql /etc/mysql /usr/share/mysql
```

6.3退出容器

```shell
exit
```

6.4找到配置文件夹

```shell
cd /mydata/
cd mysql/
cd conf/
```

6.5创建msyql配置文件 my.cnf

```shell
[mysql]
#设置mysql客户端默认字符集
default-character-set=UTF8MB4
[mysqld]
#设置3306端口
port=3306
#配置服务器的服务号，具备日后需要集群做准备
server-id = 1
#开启MySQL数据库的二进制日志，用于记录用户对数据库的操作SQL语句，具备日后需要集群做准备
log-bin=mysql-bin
#允许最大连接数
max_connections=200
#允许连接失败的次数
max_connect_errors=10
#默认使用“mysql_native_password”插件认证
default_authentication_plugin=mysql_native_password
#服务端使用的字符集默认为8比特编码的latin1字符集
character-set-server=UTF8MB4
#开启查询缓存
explicit_defaults_for_timestamp=true
#创建新表时将使用的默认存储引擎
default-storage-engine=INNODB
#等待超时时间秒
wait_timeout=60
#交互式连接超时时间秒
interactive-timeout=600
# 对数据库表大小写不敏感设置，默认设置为小写，比较也全部设置为小写在比较
lower-case-table-names=1
# 设置默认时区
default-time_zone='+8:00'
#解决 only_full_group_by 问题
sql_mode='STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_ENGINE_SUBSTITUTION'
```

6.6重启容器

```shell
docker restart msyql
docker exec -it mysql /bin/bash
```

6.7进入文件夹

```shell
cd /etc/mysql/
cat my.cnf
```

7.MySQL外部链接

```shell
IP：虚拟机IP地址——云主机地址
端口：
名称：
密码：
```

配置文件参考地址：https://blog.csdn.net/donkor_/article/details/139879575

















# Docker部署MySQL 8.3.0

## 一、环境

| 名称         | 备注                  |
| ------------ | --------------------- |
| 操作命令对象 | Docker 版本26.1.3     |
| 远程操作系统 | CentOS 7.9 64位       |
| 远程管理工具 | FinalShell 版本4.3.11 |
| 安装软件名称 | MySQL 版本8.3.0       |

## 二、拉取镜像

### 2.1 查找 Docker Hub 上的 MySQL 镜像

```shell
docker search mysql
```

![](https://www.codepeople.cn/imges/00x168.png)

### 2.2 拉取MySQL镜像

由于我本地用的是8.3.0的，所以服务器也拉取`tag`8.3.0的版本

```shell
docker pull mysql:8.3.0
```

![](https://www.codepeople.cn/imges/00x169.png)

### 2.3 查看MySQL镜像

```shell
docker images mysql:8.3.0
```

![](https://www.codepeople.cn/imges/00x170.png)

## 三、在宿主机创建目录

### 3.1 创建挂载目录

后面用于挂载`mysql容器`内目录，这里就放在home目录下

```shell
mkdir -p  /home/mysql/{conf,data,log}
```

![](https://www.codepeople.cn/imges/00x171.png)

### 3.2 创建配置文件

```shell
cd /home/mysql/conf
vim my.cnf
```

按键盘`i`键进行输入

```shell
[client]
#设置客户端默认字符集utf8mb4
default-character-set=utf8mb4
[mysql]
#设置服务器默认字符集为utf8mb4
default-character-set=utf8mb4
[mysqld]
#设置3306端口
port=3306
#配置服务器的服务号，具备日后需要集群做准备
server-id = 1
#开启MySQL数据库的二进制日志，用于记录用户对数据库的操作SQL语句，具备日后需要集群做准备
log-bin=mysql-bin
#设置清理超过30天的日志，以免日志堆积造过多成服务器内存爆满。2592000秒等于30天的秒数
binlog_expire_logs_seconds = 2592000
#解决MySQL8.0版本GROUP BY问题
sql_mode='STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_ENGINE_SUBSTITUTION'
#允许最大的连接数
max_connections=1000
# 禁用符号链接以防止各种安全风险
symbolic-links=0
# 设置东八区时区
default-time_zone = '+8:00'
```

按键盘`esc`键退出输入，输入`:wq`保存退出

![](https://www.codepeople.cn/imges/00x172.png)



## 四、启动MySQL容器

`-p`表示端口映射
`--restart=always`表示容器退出时总是重启
`--name`表示容器命名
`--privileged=true`表示赋予容器权限修改宿主文件权利
`-v /home/mysql/log:/var/log/mysql`表示容器日志挂载到宿主机
`-v /home/mysql/data:/var/lib/mysql`表示容器存储文件挂载到宿主机
`-v /home/mysql/conf/my.cnf:/etc/mysql/my.cnf`表示容器配置文件挂载到宿主机
`-e MYSQL_ROOT_PASSWORD=a12bCd3_W45pUq6`表示设置`mysql`的`root`用户密码,建议用强密码
`-d`表示后台运行

```shell
docker run \
-p 3306:3306 \
--restart=always \
--name mysql \
--privileged=true \
-v /home/mysql/log:/var/log/mysql \
-v /home/mysql/data:/var/lib/mysql \
-v /home/mysql/conf/my.cnf:/etc/mysql/my.cnf \
-e MYSQL_ROOT_PASSWORD=a12bCd3_W45pUq6 \
-d mysql:8.3.0
```

![](https://www.codepeople.cn/imges/00x173.png)

五、测试
使用Navicat进行连接，因为我服务器配置了ssh的密钥连接方式，所以截图多了一个ssh server。

服务器如何生成密钥对，服务器配置公钥，本地配置私钥，Navicat使用ssh隧道连接mysql数据库，可以查看如下2篇文章

服务器生成密钥对、服务器配置公钥、本地配置私钥
Navicat使用ssh隧道连接mysql数据库
![](https://www.codepeople.cn/imges/00x174.png)



文章来源：https://blog.csdn.net/donkor_/article/details/139879575
