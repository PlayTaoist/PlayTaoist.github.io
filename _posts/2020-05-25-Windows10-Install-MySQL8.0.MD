---
title:  "Windows10 安装mysql 8.0.20 winx64.zip压缩版安装配置方法图文教程"
categories: Windows10 MySQL8.0
tags: Windows10 MySQL8.0
author: LIUREN
---

# Windows10 安装mysql 8.0.20 winx64.zip压缩版安装配置方法图文教程

> 这篇文章主要为大家详细介绍了mysql 8.0.20 winx64.zip压缩版安装配置方法图文教程，文中示例代码介绍的非常详细，具有一定的参考价值，感兴趣的小伙伴们可以参考一下。该方法主要是提供Window用户用来绿色版本的安装步骤。主要是因为exe版本安装会装好多不必要的程序，特别是有些需要开机启动很讨厌，会影响开机速度，所以记录了一篇【绿色版】的安装方法
>
> 协议：CC BY-SA 4.0 https://creativecommons.org/licenses/by-sa/4.0/  
>
> 版权声明：本文为原创文章，遵循 CC 4.0 BY-SA 版权协议，转载请附上原文出处链接及本声明。



## 第一步：下载MySQL

下载地址如下：<https://dev.mysql.com/downloads/mysql/>

![](https://www.codepeople.cn/imges/00x19.png)

![](https://www.codepeople.cn/imges/00x20.png)

## 第二步：在F盘新建mysql目录，在mysql下新建MySQL文件夹，将压缩包解压到MySQL文件夹中。

![](https://www.codepeople.cn/imges/00x21.png)

## 第三步：在zip解压根目录下新建my.ini。（新建文本文档，并将后缀名改成.ini）

![](https://www.codepeople.cn/imges/00x22.png)

## 第四步：然后我们编辑my.ini，设置MySQL根目录，以及数据库数据存放的目录等。

```ini
[mysqld]
# 设置3306端口
port=3306
# 设置mysql的安装目录
basedir=F:\mysql\MySQL
# 设置mysql数据库的数据的存放目录
datadir=F:\mysql\Data
# 允许最大连接数
max_connections=200
# 允许连接失败的次数。这是为了防止有人从该主机试图攻击数据库系统
max_connect_errors=10
# 服务端使用的字符集默认为UTF8MB4
character-set-server=UTF8MB4
# 创建新表时将使用的默认存储引擎
default-storage-engine=INNODB
# 默认使用“mysql_native_password”插件认证
default_authentication_plugin=mysql_native_password
[mysql]
# 设置mysql客户端默认字符集
default-character-set=UTF8MB4
[client]
# 设置mysql客户端连接服务端时默认使用的端口
port=3306
default-character-set=UTF8MB4
```

![](https://www.codepeople.cn/imges/00x23.png)

## 第五步：避免权限问题出错我们尽量使用管理员身份运行CMD，否则在安装时会报错，会导致安装失败的情况。以管理员打开命令行并进入到解压根目录/bin目录下

![](https://www.codepeople.cn/imges/00x24.png)

![](https://www.codepeople.cn/imges/00x25.png)

## 第六步：初始化数据库，指令：

```powershell
mysqld --initialize
```

![](https://www.codepeople.cn/imges/00x26.png)

## 第七步：获取初始化数据库随机密码。执行完上一步之后，在Data目录下生的文件有一个.err文件，这里面有初始化的密码。我们编辑打开此文件，找到密码。该文件命名规则是【电脑用户名.err】如下图

![](https://www.codepeople.cn/imges/00x27.png)

![](https://www.codepeople.cn/imges/00x28.png)

## 第八步：安装MySQL服务

```powershell
mysqld --install mysql80
```

![](https://www.codepeople.cn/imges/00x29.png)

如果需要卸载mysql服务

```powershell
mysqld --remove mysql80
```



## 第九步：启动服务

```powershell
net start mysql80
```

![](https://www.codepeople.cn/imges/00x30.png)

## 第十步：修改密码

```powershell
mysql -u root -p
```

回车输入刚才生成的临时密码：

![](https://www.codepeople.cn/imges/00x31.png)

修改

```powershell
ALTER user 'root'@'localhost' IDENTIFIED BY '123456';
```

![](https://www.codepeople.cn/imges/00x32.png)

# 第十一步：navicat连接测试

不同的版本安装连接如下：

[mysql不同版本安装教程](https://www.jb51.net/Special/832.htm)

[mysql5.7各版本安装教程](https://www.jb51.net/Special/863.htm)

[mysql5.6各版本安装教程](https://www.jb51.net/Special/914.htm)



博客地址：<https://www.codepeople.cn>

=====================================================================

微信公众号：

![](https://www.codepeople.cn/imges/weixin_icon/weixin.jpg)