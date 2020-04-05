---
title:  "CentOS7 多台电脑使用NTP进行时间同步"
categories: CentOS7
tags: CentOS7
author: LIUREN
---



## CentOS7 多台电脑使用NTP进行时间同步

> NTP在linux下有两种时钟同步方式,分别为直接同步和平滑同步: 
> **直接同步** 
> 使用ntpdate命令进行同步,直接进行时间变更。如果服务器上存在一个12点运行的任务,当前服务器时间是13点,但标准时间时11点,使用此命令可能会造成任务重复执行。因此使用ntpdate同步可能会引发风险,因此该命令也多用于配置时钟同步服务时第一次同步时间时使用。 
> **平滑同步** 
> 使用ntpd进行时钟同步,可以保证一个时间不经历两次,它每次同步时间的偏移量不会太陡,是慢慢来的,这正因为这样,ntpd平滑同步可能耗费的时间比较长。
>
> **标准时钟同步服务** 
> http://www.pool.ntp.org/zone/cn网站包含全球的标准时间同步服务,也包括对中国时间的同步,对应的URL为cn.pool.ntp.org,在其中也描述了ntp配置文件中的建议写法: 
> server 1.cn.pool.ntp.org 
> server 3.asia.pool.ntp.org 
> server 2.asia.pool.ntp.org



## 1.**NTP时钟同步方式说明**

NTP在linux下有两种时钟同步方式,分别为直接同步和平滑同步: 
**直接同步** 
使用ntpdate命令进行同步,直接进行时间变更。如果服务器上存在一个12点运行的任务,当前服务器时间是13点,但标准时间时11点,使用此命令可能会造成任务重复执行。因此使用ntpdate同步可能会引发风险,因此该命令也多用于配置时钟同步服务时第一次同步时间时使用。 
**平滑同步** 
使用ntpd进行时钟同步,可以保证一个时间不经历两次,它每次同步时间的偏移量不会太陡,是慢慢来的,这正因为这样,ntpd平滑同步可能耗费的时间比较长。

**标准时钟同步服务** 
http://www.pool.ntp.org/zone/cn网站包含全球的标准时间同步服务,也包括对中国时间的同步,对应的URL为cn.pool.ntp.org,在其中也描述了ntp配置文件中的建议写法: 
server 1.cn.pool.ntp.org 
server 3.asia.pool.ntp.org 
server 2.asia.pool.ntp.org

## 2.**环境情况**

**准备四台电脑，分别为：**

| IP             | 用途                                       |
| -------------- | ------------------------------------------ |
| 192.168.11.212 | ntpd服务器，用于与外部公共ntpd同步标准时间 |
| 172.16.248.129 | ntpd客户端，用于与ntpd同步时间             |

## 3. 检查服务是否安装

```shell
systemctl status ntpd
```

或者使用rpm检查ntp包是否安装

```shell
rpm -q ntp
```

如果已经安装则略过此步，否则使用yum进行安装，并设置系统开机自动启动并启动服务

```shell
[root@localhost kevin]# yum -y install ntp
[root@localhost kevin]# systemctl enable ntpd
[root@localhost kevin]# systemctl start ntpd
```

## 4. 设置ntp服务器: 192.168.11.212

配置前先使用命令:ntpdate -u cn.pool.ntp.org,同步服务器 
-修改/etc/ntp.conf文件,红色字体是修改的内容 

```shell
#For more information about this file, see the man pages 
#ntp.conf(5), ntp_acc(5), ntp_auth(5), ntp_clock(5), ntp_misc(5), ntp_mon(5). 
driftfile /var/lib/ntp/drift 
#Permit time synchronization with our time source, but do not 
#permit the source to query or modify the service on this system. 

restrict default nomodify notrap nopeer noquery

# Permit all access over the loopback interface. This could 
# be tightened as well, but to do so would effect some of 
# the administrative functions. 
restrict 127.0.0.1 
restrict ::1

# Hosts on local network are less restricted. 
#restrict 192.168.1.0 mask 255.255.255.0 nomodify notrap 
restrict 172.16.248.0 mask 255.255.255.0 nomodify notrap

# Use public servers from the pool.ntp.org project. 
# Please consider joining the pool (http://www.pool.ntp.org/join.html). 
#server 0.centos.pool.ntp.org iburst 

server 2.cn.pool.ntp.org 
server 1.asia.pool.ntp.org 
server 2.asia.pool.ntp.org

#broadcast 192.168.1.255 autokey # broadcast server 
#broadcastclient # broadcast client 
#broadcast 224.0.1.1 autokey # multicast server 
#multicastclient 224.0.1.1 # multicast client 
#manycastserver 239.255.254.254 # manycast server 
#manycastclient 239.255.254.254 autokey # manycast client 

#允许上层时间服务器主动修改本机时间 

restrict 2.cn.pool.ntp.org nomodify notrap noquery 
restrict 1.asia.pool.ntp.org nomodify notrap noquery 
restrict 2.asia.pool.ntp.org nomodify notrap noquery

server 127.0.0.1 # local clock 
fudge 127.0.0.1 stratum 10

# Enable public key cryptography. 
#crypto

includefile /etc/ntp/crypto/pw

# Key file containing the keys and key identifiers used when operating 
# with symmetric key cryptography. 
keys /etc/ntp/keys

# Specify the key identifiers which are trusted. 
#trustedkey 4 8 42

# Specify the key identifier to use with the ntpdc utility. 
#requestkey 8

# Specify the key identifier to use with the ntpq utility. 
#controlkey 8

# Enable writing of statistics records. 
#statistics clockstats cryptostats loopstats peerstats

# Disable the monitoring facility to prevent amplification attacks using ntpdc 
# monlist command when default restrict does not include the noquery flag. See 
# CVE-2013-5211 for more details. 
# Note: Monitoring will not be disabled with the limited restriction flag. 
disable monitor
```

修改完成后重启ntpd服务`systemctl restart ntpd`

使用`ntpq -p `查看网络中的NTP服务器,同时显示客户端和每个服务器的关系

使用ntpstat 命令查看时间同步状态,这个一般需要5-10分钟后才能成功连接和同步。所以,服务器启动后需要稍等下: 
刚启动的时候,一般是:

`# ntpstat unsynchronised time server re-starting polling server every 64 s`

连接并同步后:

`# ntpstat synchronised to NTP server (202.112.10.36) at stratum 3 time correct to within 275 ms polling server every 256 s`

## **5. 设置ntp客户端: 172.16.248.129**

安装ntp服务并设置为自动启动,和前面的设置方式相同。然后编辑/etc/ntp.conf文件,红色字体为变化的内容。

```shell
# For more information about this file, see the man pages 
# ntp.conf(5), ntp_acc(5), ntp_auth(5), ntp_clock(5), ntp_misc(5), ntp_mon(5).

driftfile /var/lib/ntp/drift

# Permit time synchronization with our time source, but do not 
# permit the source to query or modify the service on this system. 
restrict default nomodify notrap nopeer noquery

# Permit all access over the loopback interface. This could 
# be tightened as well, but to do so would effect some of 
# the administrative functions. 
restrict 127.0.0.1 
restrict ::1

# Hosts on local network are less restricted. 
#restrict 192.168.1.0 mask 255.255.255.0 nomodify notrap

# Use public servers from the pool.ntp.org project. 
# Please consider joining the pool (http://www.pool.ntp.org/join.html). 
#server 0.centos.pool.ntp.org iburst 

server 172.16.248.1 

restrict 172.16.248.1 nomodify notrap noquery 

server 127.0.0.1 
fudge 127.0.0.1 stratum 10

#broadcast 192.168.1.255 autokey # broadcast server 
#broadcastclient # broadcast client 
#broadcast 224.0.1.1 autokey # multicast server 
#multicastclient 224.0.1.1 # multicast client 
#manycastserver 239.255.254.254 # manycast server 
#manycastclient 239.255.254.254 autokey # manycast client

# Enable public key cryptography. 
#crypto 
includefile /etc/ntp/crypto/pw

# Key file containing the keys and key identifiers used when operating 
# with symmetric key cryptography. 
keys /etc/ntp/keys

# Specify the key identifiers which are trusted. 
#trustedkey 4 8 42

# Specify the key identifier to use with the ntpdc utility. 
#requestkey 8

# Specify the key identifier to use with the ntpq utility. 
#controlkey 8

# Enable writing of statistics records. 
#statistics clockstats cryptostats loopstats peerstats

# Disable the monitoring facility to prevent amplification attacks using ntpdc 
# monlist command when default restrict does not include the noquery flag. See 
# CVE-2013-5211 for more details. 
# Note: Monitoring will not be disabled with the limited restriction flag. 
disable monitor
```

重启ntpd服务 

`#systemctl restart ntpd`

启动后,查看同步情况 

`# ntpq -p `
`# ntpstat`

因为是内网,一般ntpstat很快就可以同步上。

参考来源:<https://www.aliyun.com/jiaocheng/152708.html>

参考来源：<https://blog.csdn.net/to_baidu/article/details/52562226>

参考来源：<https://www.cnblogs.com/betajj-blog/p/5587202.html>



## 另一种方案参考

> 同一个程序运行在多台服务器上时，有时需要这些服务器的时钟相同，但是当服务器运行一定时间之后，各个服务器上的时钟会有一定差异，此时就需要让所有服务器自动校准各自时钟。
>
> NTP是一个很方便的时钟服务，可以用此来进行时钟同步。

**（以下解决应用于不连接外网的局域网，需求是同步所有服务器时钟。）**

### 1.选定其中一台服务器作为基准服务器，即提供时钟服务。

（1）首先确定ntp是否存在，若不存在则通过以下命令安装：

```shell
yum install ntp
```

（2）修改配置文件/etc/ntp.conf，在其中插入：

```shell
restrict 192.168.20.0 mask 255.255.255.0 nomodify notrap  
```

其中：

用restrict控管权限
nomodify - 用户端不能更改ntp服务器的时间参数
noquery - 用户端不能使用ntpq，ntpc等命令来查询ntp服务器
notrap - 不提供trap远端登陆

（3）此时可以向192.168.20.0/24的机器提供时间服务了，开启ntp服务：

```shell
# 启动ntp服务
systemctl start ntpd
# 让ntp服务开机启动
systemctl enable ntpd
```

（4）检查ntp端口是否已经开启：

```shell
netstat -unlnp  
```

如果正常，应该可以看到123端口已经开启。

### 2.客户端同步（使用ntpd服务）

（1）修改/etc/ntp.conf，在里边插入：

```shell
server 192.168.20.91  
```

（2）使ntpd服务开启启动

```shell
systemctl start ntpd
```

参考文章:<https://blog.csdn.net/tomspcc/article/details/52944868>

参考文章:<https://blog.csdn.net/yuanfang_way/article/details/53959591>