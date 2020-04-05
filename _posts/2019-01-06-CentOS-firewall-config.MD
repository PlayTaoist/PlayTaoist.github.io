---
title:  "CentOS7防火墙管理"
categories: CentOS7
tags: CentOS7
author: LIUREN
---



## CentOS7防火墙管理

> 首先确定查询一下Linux版本号：命令：`cat /etc/redhat-release`  
>
> 如果想要操作防火墙记得使用管理员账户如何切换到管理员账号：
>
> `su root` 然后输入密码,如何切换回普通用户`su ucenter`



## 1.防火墙状态firewalld

### 查看防火墙状态

启动：` systemctl start firewalld`

关闭：` systemctl stop firewalld`

查看状态： `systemctl status firewalld `

开机禁用  ：` systemctl disable firewalld`

开机启用  ： `systemctl enable firewalld`

### 那怎么开启一个端口呢

添加:`firewall-cmd --zone=public --add-port=80/tcp --permanent `

重新载入:`firewall-cmd --reload`

查看:`firewall-cmd --zone= public --query-port=80/tcp`

删除:`firewall-cmd --zone= public --remove-port=80/tcp --permanent`

查看所有打开的端口： `firewall-cmd --zone=public --list-ports`

查看所有配置： `firewall-cmd --list-all`

指定IP与端口: `firewall-cmd --permanent --add-rich-rule="rule family="ipv4" source address="192.168.142.166" port protocol="tcp" port="5432" accept"`

指定IP段访问：`firewall-cmd --permanent --add-rich-rule="rule family="ipv4" source address="192.168.2.0/24" port protocol="tcp" port="5432" accept"` 

删除规则：`firewall-cmd --permanent --remove-rich-rule="rule family="ipv4" source address="192.168.142.166" port protocol="tcp" port="11300" accept"`



## 2.防火墙iptables

> 首先执行`systemctl status iptables`查看是否有防火墙，如果有是否开启：
>
> 结果如下：`Unit iptables.service could not be found` 没有iptables防火墙

```shell
systemctl start iptables #启动

systemctl status iptables #查看运行状态

systemctl restart iptables.service #重启

systemctl stop iptables.service #停止

systemctl enable iptables.service #设置开机启动

systemctl disable iptables.service #禁止开机启动
```

常见案例：

##### 禁止192.168.1.3 IP地址的所有类型数据接入

```shell
iptables -A INPUT ! -s 192.168.1.3 -j DROP
```

##### 开放端口

```shell
iptables -A INPUT -p tcp --dport 80 -j ACCEPT #开放80端口
```

##### 开放端口范围

```shell
iptables -I INPUT -p tcp --dport 22:80 -j ACCEPT #开发22-80范围的端口
```

##### 不允许80端口流出

```shell
iptables -I OUTPUT -p tcp --dport 80 -j DROP
```

## 3.设置系统时间同步

#### 使用`timedatectl`命令

##### 显示当前日期和时间： 

```shell
timedatectl
```

改变ntpd和chrony状态不会立即生效，需要执行：

```shell
systemctl restart systemd-timedated.service
```

##### 改变当前时间

  该命令同时更新系统时间和硬件时钟。其结果类似于使用`date --set`和`hwclock --systohc`命令。如果`NTP`启用服务，该命令将失败。

要将当前日期更改为2017年6月2日并保持当前时间（下午11:26）

```shell
timedatectl set-time "2019-01-16 12:00:00"
```

##### 更改时区

列出时区

```shell
timedatectl list-timezones|grep Asia
```

更改时区为上海

```shell
timedatectl set-timezone Asia/Shanghai
```

使用date更改时间

要将当前日期更改为2017年6月2日并保持当前时间（下午11:26）

```shell
date -s "2019-01-16 12:00:00"
```

#### 启动自动同步时间

```shell
#yes或no; 1或0也可以
timedatectl set-ntp yes
```



#### 使用hwclock

**显示硬件时钟当前时间**

```shell
hwclock
```

**将硬件时钟设置为特定日期和时间,并将硬件时钟保持为UTC**

```shell
hwclock --set --date "21 Oct 2016 21:17" --utc
```

**硬件时钟与系统时间同步，并设置为UTC时间**

```shell
hwclock --systohc --utc
```

**硬件时钟与系统时间同步，并设置为本地时间**

```shell
hwclock --systohc --localtime
```

**注意:** 硬件时钟默认使用UTC时间，因为硬件时钟不能保存时区和夏令时调整，修改后就无法从硬件时钟中读取出准确标准时间，因此不建议修改。修改后系统会出现下面的警告。

GMT、UTC、CST、DST 时间

(1) UTC

整个地球分为二十四时区，每个时区都有自己的本地时间。在国际无线电通信场合，为了统一起见，使用一个统一的时间，称为通用协调时(UTC, Universal Time Coordinated)。

(2) GMT

格林威治标准时间 (Greenwich Mean Time)指位于英国伦敦郊区的皇家格林尼治天文台的标准时间，因为本初子午线被定义在通过那里的经线。(UTC与GMT时间基本相同，本文中不做区分)

(3) CST

中国标准时间 (China Standard Time)

(4) DST

夏令时(Daylight Saving Time) 指在夏天太阳升起的比较早时，将时钟拨快一小时，以提早日光的使用。（中国不使用）

GMT + 8 = UTC + 8 = CST