---
title:  "CentOS 中文件夹同步 采用Lsyncd对文件夹进行同步"
categories: CentOS Lsyncd
tags: CentOS Lsyncd
author: LIUREN
---

# CentOS 中文件夹同步 采用Lsyncd对文件夹进行同步

> 使用CentOS 中文件夹同步 采用Lsyncd对文件夹进行同步，集群中采用Lsyncd对上传图片进行文件同步，同步后在双击热备中，RUOYI无论访问哪一台服务器都能访问到文件夹；有两台CentOS 7服务器，其中A：`192.168.1.100`文件需要实时同步到B：`192.168.1.200`
>
> 协议：CC BY-SA 4.0 https://creativecommons.org/licenses/by-sa/4.0/  
>
> 版权声明：本文为原创文章，遵循 CC 4.0 BY-SA 版权协议，转载请附上原文出处链接及本声明。



有两台CentOS 7服务器，其中A：`192.168.1.100`文件需要实时同步到B：`192.168.1.200`

### 安装rsync

lsyncd基于rsync，且要求rsync >= 3.1，可输入`rsync -v`查看当前版本，如果不符合要求，请参考：《[CentOS升级rsync版本](https://faq.xiaoz.me/archives/182.html)》，两台服务器A/B均要安装rsync

### 设置密钥登录

如果想要将A的数据同步到B，A必须拥有免密码登录B的权限，可以设置密钥登录来完成。只需要两条命令即可，下面的命令在A服务器执行：

```
#生成密钥文件
ssh-keygen -t rsa
#将公钥拷贝到B
ssh-copy-id username@remote-server
#测试
ssh username@remote-server
```

`username`为B服务器用户名，`remote-server`为B服务器IP地址，如果服务器不是使用的`22`作为SSH端口，需要`-p`参数指定端口号。更多说明可参考：[SSH无密码登录：只需两个简单步骤 (Linux)](https://www.linuxdashen.com/ssh-key：两个简单步骤实现ssh无密码登录)

### 安装epel仓库

CentOS 7内置的源并没有包括Lsyncd，可以自行编译安装Lsyncd，不过更简单的办法是先安装epel源，就可以直接yum安装了，以下命令再A服务器执行。

```
yum -y install epel-release
```

### 安装lsyncd并配置

如果已经安装了epel仓库，直接输入下面的命令安装lsyncd即可，一下命令在A服务器执行。

```
#安装lsyncd
yum -y install lsyncd
#查看lsyncd版本
lsyncd --version
```

安装完成后配置文件位于`/etc/lsyncd.conf`，会看到如下的内容，其中`--`是注释符。

```shell

----
-- User configuration file for lsyncd.
--
-- Simple example for default rsync, but executing moves through on the target.
--
-- For more examples, see /usr/share/doc/lsyncd*/examples/
--
-- sync{default.rsyncssh, source="/var/www/html", host="localhost", targetdir="/tmp/htmlcopy/"}
```

`vi /etc/lsyncd.conf`修改配置文件，注释或删除掉默认的最后一行，然后添加自己的配置。

```shell
settings {
   logfile = "/home/lsyncd.log",
   statusFile = "/home/lsyncd.status",
   insist = true,
   statusInterval = 10
}

sync {
   default.rsyncssh,
   source = "/home/ruoyi/uploadPath/upload",
   host = "192.168.1.200",
   targetdir="/home/test/uploadPath/upload",
   rsync = {
    archive = true,
    compress = false,
    whole_file = false
   },
   ssh = {
    port = 22
   }
}
```

继续输入命令`lsyncd -nodaemon /etc/lsyncd.conf`运行查看下是否有报错，如果有报错根据报错情况检查修改，如果没有报错直接退出后输入命令：`systemctl start lsyncd`启动lsyncd即可。



### 部分参数说明

`settings`为全局配置，部分参数如下：

- logfile：日志文件路径
- statusFile：进程路径
- insist：继续运行，即使有失败的目标。
- statusInterval：多少秒写入文件，默认是10s

`sync`为同步配置，部分参数如下：

- source：本地文件目录

- host：远程服务器地址

- targetdir：远程目标目录

- port：目前主机SSH端口号，默认为22

  

```
启动
systemctl start lsyncd
#停止
systemctl stop lsyncd
#重启
systemctl restart lsyncd
#设置开机自动启动
systemctl enable lsyncd
```

### 总结

以上方法在CentOS 7系统测试，如果是CentOS 6系统可能部分地方略有不同，但流程和步骤都一样。

lsyncd是基于inotify + rsync的开源同步软件，支持增量同步，与[Resilio Sync](https://www.xiaoz.me/archives/8219)相比优势是更加安全、占用资源更低，多种参数自定义配置更加方便。劣势是配置要比Resilio Sync麻烦一点，如果多台服务器的情况下速度可能不及Resilio Sync，因为Resilio Sync使用了P2P分享。

> Lsyncd官方帮助手册：https://axkibe.github.io/lsyncd/
> 项目地址：https://github.com/axkibe/lsyncd



博客地址：<https://www.codepeople.cn>

=====================================================================

微信公众号：

![](https://www.codepeople.cn/imges/weixin_icon/weixin.jpg)