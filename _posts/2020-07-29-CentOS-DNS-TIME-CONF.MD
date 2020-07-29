---
title:  "CentOS7.6 更新DNS，并和互联网同步时间"
categories: CentOS DNS
tags: CentOS DNS
author: LIUREN
---

# CentOS7.6 更新DNS，并和互联网同步时间

> 在CentOS 7下，手工设置 /etc/resolv.conf 里的DNS，过了一会，发现被系统重新覆盖或者清除了。和CentOS 6下的设置DNS方法不同，有几种方式： 1、使用全新的命令行工具 nmcli 来设置；CentOS7.6 更新DNS，并和互联网同步时间
>
> 协议：CC BY-SA 4.0 https://creativecommons.org/licenses/by-sa/4.0/  
>
> 版权声明：本文为原创文章，遵循 CC 4.0 BY-SA 版权协议，转载请附上原文出处链接及本声明。



1、使用nmcli方法

```powershell
#显示当前网络连接
#nmcli connection show
NAME UUID                                 TYPE           DEVICE
eno1 5fb06bd0-0bb0-7ffb-45f1-d6edd65f3e03 802-3-ethernet eno1

#修改当前网络连接对应的DNS服务器，这里的网络连接可以用名称或者UUID来标识
#nmcli con mod eno1 ipv4.dns "114.114.114.114 8.8.8.8"

#将dns配置生效
#nmcli con up eno1
```

2、使用传统方法，手工修改 /etc/resolv.conf

- 修改 /etc/NetworkManager/NetworkManager.conf 文件，在main部分添加 “dns=none” 选项：

```
[main]
plugins=ifcfg-rh
dns=none
```

- NetworkManager重新装载上面修改的配置

```
# systemctl restart NetworkManager.service
```

- 手工修改 /etc/resolv.conf

```
nameserver 114.114.114.114
nameserver 8.8.8.8
```

详细参见：

```
# man NetworkManager.conf
# man nmcli
```



同步时间命令：

```shell
yum install ntpdate
ntpdate cn.pool.ntp.org
```





博客地址：<https://www.codepeople.cn>

=====================================================================

微信公众号：

![](https://www.codepeople.cn/imges/weixin_icon/weixin.jpg)