---
title:  "CentOS7安装Ansible"
categories:  Ansible CentOS7
tags: Ansible CentOS7
author: LIUREN
---

# CentOS7安装Ansible

> CentOS7安装Ansible
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



ansible自动化部署条件
1.建议基于ssh密钥方式建立远程连接
2.基于ssh口令方式建立远程连接（不建议）

**安装sshpass**

```shell
yum install sshpass -y
```

**生成密钥**

```shell
[root@m01 ~]# ssh-keygen -t dsa -f /root/.ssh/id_dsa -N ""
Generating public/private dsa key pair.
Created directory '/root/.ssh'.
Your identification has been saved in /root/.ssh/id_dsa.
Your public key has been saved in /root/.ssh/id_dsa.pub.
The key fingerprint is:
SHA256:gfr8/bG2IAzxNJiom7WGwba8G26BZ5yfxJMp6O3Ouh4 root@m01
The key's randomart image is:
+---[DSA 1024]----+
|                 |
|     . +         |
|    . = +        |
| . . . + o       |
| +=ooo. S        |
|ooBB*+ o         |
|.EO=ooo o . .    |
| o+=o  . o ..o   |
|.=O=    . .o+.   |
+----[SHA256]-----+
```

**分发密钥**

```shell
# 免交互式批量分发公钥脚本
[root@m01 ~]# vim ~/ssh-fenfa.sh
#!/bin/bash
rm -f /root/.ssh/id_dsa 
ssh-keygen -t dsa -f /root/.ssh/id_dsa -N ""
  for ip in 7 8 
do
sshpass -p123456 ssh-copy-id -i /root/.ssh/id_dsa.pub "-o StrictHostKeyChecking=no" 10.4.7.$ip
done

# 执行脚本
[root@m01 ~]# sh ~/ssh-fenfa.sh
```

**一键ssh登录测试for循环**

```shell
[root@m01 ~]# for i in 7 8 ;do ssh 10.4.7.$i  date ;done
Mon May  9 15:49:32 CST 2022
Mon May  9 15:49:32 CST 2022
```



**安装Ansible**

Centos7 yum安装:Ansible软件默认不在标准仓库中，需要用到repo源

**管理机器上安装**

```shell
#添加repo
[root@m01 ~]# yum install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm

#yum安装ansilbe
[root@m01 ~]# yum install ansible -y
[root@m01 ~]# rpm -qa ansible

#检查ansible版本
[root@m01 ~]# ansible --version
ansible 2.9.27
  config file = /etc/ansible/ansible.cfg
  configured module search path = [u'/root/.ansible/plugins/modules', u'/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/lib/python2.7/site-packages/ansible
  executable location = /usr/bin/ansible
  python version = 2.7.5 (default, Nov 16 2020, 22:23:17) [GCC 4.8.5 20150623 (Red Hat 4.8.5-44)]
```

**添加主机清单**

```shell
[root@m01 ~]# vim /etc/ansible/hosts
[sa]
10.4.7.7
10.4.7.8
```

**测试ansible**:ping模块用于测试ansible与被受控端的连通性

```shell
[root@m01 ~]# ansible sa -m ping
10.4.7.8 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": false,
    "ping": "pong"
}
10.4.7.7 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": false,
    "ping": "pong"
}
```

**安装完成**

=====================================================================

微信公众号：

![](https://www.codepeople.cn/imges/weixin_icon/weixin.jpg)