---
title:  "CentOS7查杀nanoWath挖矿木马"
categories: Nacos
tags: Nacos
author: LIUREN
---

# CentOS7查杀nanoWath挖矿木马

> H2和derby作为一个内置数据库感觉非常好玩。上一篇讲了Springboot集成Nacos，当Nacos是单独运行未配置任何信息的时候，使用的数据库是derby；受到启发，如果我们自己做的一个小应用不想装其他什么的mysql这种大的。单单就是为了记录一下日志信息。我们可以考虑内置数据库H2.现在为大家做一篇整合过程，这个是在Springboot+Nacos基础上弄的。可称为Springboot + Nacos + H2 + JPA
>
> 协议：CC BY-SA 4.0 https://creativecommons.org/licenses/by-sa/4.0/  
>
> 版权声明：本文为原创文章，遵循 CC 4.0 BY-SA 版权协议，转载请附上原文出处链接及本声明。



## 该信息是copy过来的。源文章的地址。

https://www.dazhuanlan.com/2020/03/24/5e7965f165c12/

![img](https://www.dazhuanlan.com/webchat.jpg)

前几天在查看服务器时状态时发现cpu占用率达到了103%

![img](http://imagebed.shaoyong.online/images/2019_7_21/1.png)

这个VPS主要是拿来玩玩，平时上面并没有什么大的服务再运行，不可能会占满CPU的；用shell客户端连上VPS，执行`ps aux`，其中一个叫nanoWatch的进程cpu占用率达到了99.5%

![img](http://imagebed.shaoyong.online/images/2019_7_21/2.png)

直接杀掉这个进程`kill -9 17305`,再检查进程，没发现异常

过了没几分钟，cpu占用又满了，查看进程，这个进程又出现了；

找到`/tmp/nanoWatch`这个文件，删掉这个文件，再杀掉进程，过了几分钟，进进程再次出现。

我怀疑是定时启动的任务， `crontab -l`查看cron计划任务，显示出来了两个定时任务，每5分钟和7分钟执行一次下载文件，我手动把这个文件下下来，是一个脚本

![img](http://imagebed.shaoyong.online/images/2019_7_21/3.png)

删除定时任务`crontab -r`，删掉文件，杀掉进程，~~结束了~~。

![img](http://imagebed.shaoyong.online/images/2019_7_21/4.png)

------

过了两天，又出现CPU占用异常。

![img](http://imagebed.shaoyong.online/images/2019_7_21/5.png)

我之前一直是用root用户并使用20端口进行ssh连接，这是非常不安全的行为，很容易被端口扫描暴力破解密码登录服务器。简单的解决办法如下

[TOC]

用 root 用户进入 `/etc/ssh/`

```
cd /etc/ssh/

vim sshd_config
#添加端口20022（22号是默认端口，注释掉也是默认开启的），并解开22端口注释
```

![img](http://imagebed.shaoyong.online/images/2019_7_21/6.png)

```
#重启ssh 
systemctl restart sshd
#查看防火墙规则
firewall-cmd --permanent --list-port
#向防火墙中添加端口
firewall-cmd --zone=public --add-port=20022/tcp --permanent
reaload 
#重新加载防火墙规则
firewall-cmd --reload
```

![img](http://imagebed.shaoyong.online/images/2019_7_21/7.png)

```
#查看20022端口是否添加成功
firewall-cmd --zone=public --query-port=20022/tcp
```

![img](http://imagebed.shaoyong.online/images/2019_7_21/8.png)

断开ssh连接，换用`20022`端口连接
修改`/etc/ssh/sshd_config`将`22`号端口注释掉

![img](http://imagebed.shaoyong.online/images/2019_7_21/9.png)

```
#重启ssh
systemctl restart sshd
#查看ssh监听的端口
ss -tnlp|grep ssh
```

![img](http://imagebed.shaoyong.online/images/2019_7_21/10.png)

# 禁用root登入，使用普通用户登入

```
#先创建一个普通用户test,同时给test用户设置密码
adduser test
passwd test
#接着禁用root登录(修改sshd_config文件)
vi /etc/ssh/sshd_config
```

![img](http://imagebed.shaoyong.online/images/2019_7_21/11.png)

```
#重启ssh
Systemctl restart sshd.service
```

------

=====================================================================

微信公众号：

![](https://www.codepeople.cn/imges/weixin_icon/weixin.jpg)