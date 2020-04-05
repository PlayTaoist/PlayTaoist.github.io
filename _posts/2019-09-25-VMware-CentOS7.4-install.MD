---
title:  "VMware下载安装CentOS7.4"
categories: CentOS7
tags: CentOS7
author: LIUREN
---

# VMware下载安装CentOS7.4

> VMware下载安装CentOS7.4，主要是通过NAT可以直接和本地机进行通信。直接通过xshell连接上服务器。
>
> 整个的安装步骤都是一步一步截图来的。整个过程主要参考的网站是CSDN:原文链接：https://blog.csdn.net/qq_39135287/article/details/83993574



###  一、CentOS7.4下载

官网下载地址：<http://vault.centos.org/>

1、进入CentOS下载官网，找到CentOS7.4版本

![](https://www.codepeople.cn/imges/centos/install/001.png)

 2、在CentOS7.4版本页面中，找到isos/

![](https://www.codepeople.cn/imges/centos/install/002.png)

 3、进入页面后，可以看到x86_64

![](https://www.codepeople.cn/imges/centos/install/003.png)

 4、在CentOS下载页面中，选择 CentOS-7-x86_64-DVD-1708.torrent进行下载

![](https://www.codepeople.cn/imges/centos/install/004.png)

5、下载完成之后，由于“CentOS-7-x86_64-DVD-1708.torrent”只是一个BT种子文件，而且非常小，这就需要我们使用迅雷等工具来对源镜像进行下载了

   （1）打开迅雷，新建任务，把“CentOS-7-x86_64-DVD-1708.torrent”BT种子文件拖入到新建任务中，并点击立即下载

![](https://www.codepeople.cn/imges/centos/install/005.jpg)

（2）下载完成之后会生成如下的几个文件，CentOS-7-x86_64-DVD-1708.iso就是我们在工作中使用的镜像了

![](https://www.codepeople.cn/imges/centos/install/006.png)

### 二、CentOS7.4安装

 1、打开你的VMware Workstation Pro，并点击“创建新的虚拟机”，没有安装VMware Workstation Pro请点击[VMware Workstation 14中文破解版 下载与安装(附密钥)](https://blog.csdn.net/qq_39135287/article/details/82924547)

![](https://www.codepeople.cn/imges/centos/install/007.png)

2、点选典型(推荐)(T)，并点击“下一步”

![](https://www.codepeople.cn/imges/centos/install/008.jpg)

3、点选稍后安装操作系统(S)，并点击“下一步”

![](https://www.codepeople.cn/imges/centos/install/009.jpg)

4、点选Linux(L)，因为我们之前下载的 CentOS-7-x86_64-DVD-1708.iso 是64位 7.4版本的，所以这里我们选择CentOS 7 64位，并点击“下一步”

![](https://www.codepeople.cn/imges/centos/install/010.jpg)

5、虚拟机名称可以更改也可以不更改看自己需求，修改虚拟机的安装路径，并点击“下一步”

![](https://www.codepeople.cn/imges/centos/install/011.jpg)

6、磁盘选择默认为20.0GB，点选将虚拟磁盘存储为单个文件(O)，并点击“下一步”

![](https://www.codepeople.cn/imges/centos/install/012.png)

7、点击“完成”

![](https://www.codepeople.cn/imges/centos/install/013.png)

8、点击“编辑虚拟机设置”

![](https://www.codepeople.cn/imges/centos/install/014.png)

9、点选“使用ISO映像文件(M)”，并添加我们之前下载好的CentOS-7-x86_64-DVD-1708.iso

![](https://www.codepeople.cn/imges/centos/install/015.png)

10、点选“自定义(U)：特定虚拟网络”，并选择VMnet8 (NAT 模式)

![](https://www.codepeople.cn/imges/centos/install/016.png)

11、选择USB 控制器，并点击“移除(R)”

![](https://www.codepeople.cn/imges/centos/install/017.png)

12、选择声卡，并点击“移除(R)”

![](https://www.codepeople.cn/imges/centos/install/018.png)

13、选择打印机，并点击“移除(R)”，最后点击“确定”

![](https://www.codepeople.cn/imges/centos/install/019.png)

14、点击“开启此虚拟机”

![](https://www.codepeople.cn/imges/centos/install/020.png)

15、正在开启虚拟机，鼠标移入到虚拟机中，并按下“↑”键，选择Install CentOS 7，最后按下“Enter 键”

        提示：  鼠标移动到虚拟机内部单击或者按下Ctrl + G，鼠标即可移入到虚拟机中
    
                     按下Ctrl + Alt，鼠标即可移出虚拟机
    
        注意：  在虚拟机中的操作，鼠标必须要移入到虚拟机中，否则虚拟机感应不到，无法对其进行操作
![](https://www.codepeople.cn/imges/centos/install/021.png)

16、按下“Enter 键”

![](https://www.codepeople.cn/imges/centos/install/022.png)

![](https://www.codepeople.cn/imges/centos/install/023.png)

17、默认安装过程中的安装界面使用English (英语)，点击“Continue”

![](https://www.codepeople.cn/imges/centos/install/024.png)

18、配置时区 (DATE & TIME)

​       （1）选择DATE & TIME

![](https://www.codepeople.cn/imges/centos/install/025.png)

​      （2）时区设置为 Region：Asia    City：Shanghai

​                日期和时间 设置与自己的电脑时间同步，最后点击“Done”

![](https://www.codepeople.cn/imges/centos/install/026.png)

19、设置软件选择 (SOFTWARE SELECTION)

​         （1）选择SOFTWARE SELECTION

![](https://www.codepeople.cn/imges/centos/install/027.png)

​	   (2）点击勾选 Compatibility Libraries 和 Development Tools

![](https://www.codepeople.cn/imges/centos/install/028.png)

20、设置安装位置 (INSTALLATION DESTINATION)

​      （1）选择INSTALLATION DESTINATION

![](https://www.codepeople.cn/imges/centos/install/029.png)

​      （2）点选 I will configuire parttioning，然后再点击“Done”

![](https://www.codepeople.cn/imges/centos/install/030.png)

​      （3）更改模式，标准分区Standard Partition，点击“+”按钮添加分区

![](https://www.codepeople.cn/imges/centos/install/031.png)

​      （4）添加 /boot分区，大小300MB，Add mount point

![](https://www.codepeople.cn/imges/centos/install/032.png)

![](https://www.codepeople.cn/imges/centos/install/033.png)

​        (5）添加 swap分区，一般情况是物理内存的2倍大小，用于物理内存不足时使用，但可能造成系统不稳定，所以看情况，可以设置小一点，甚至设置为0MB，这里我设置为512MB，Add mount point

![](https://www.codepeople.cn/imges/centos/install/034.png)

![](https://www.codepeople.cn/imges/centos/install/035.png)

​     （6）增加根分区，不填写大小，即默认剩余的空间都给根分区，Add mount point

![](https://www.codepeople.cn/imges/centos/install/036.png)

![](https://www.codepeople.cn/imges/centos/install/037.png)

​     （7）点击“Done”

![](https://www.codepeople.cn/imges/centos/install/038.png)

​     （8）点击“Accept Changes”

![](https://www.codepeople.cn/imges/centos/install/039.png)

21、点击“Begin Installation”，开始安装

![](https://www.codepeople.cn/imges/centos/install/040.png)

22、设置系统用户root的密码 (ROOT PASSWORD)

​      （1）选择ROOT PASSWORD

![](https://www.codepeople.cn/imges/centos/install/041.png)

​      （2）为root设置密码 (密码长度最好不要小于6位数)，然后点击“Done”

![](https://www.codepeople.cn/imges/centos/install/042.png)

23、等待安装完成，然后点击“Reboot”

![](https://www.codepeople.cn/imges/centos/install/043.png)

### 三、CentOS7.4基本设置

 1、修改网卡名称

​     （1）进入到network-scripts目录下，把ifcfg-ens32 重命名为 ifcfg-eth0

```shell
cd /etc/sysconfig/network-scripts/    #进入到network-scripts目录下   
mv ifcfg-ens32 ifcfg-eth0            #重命名为ifcfg-eth0
```

![](https://www.codepeople.cn/imges/centos/install/044.png)

​    （2）编辑 ifcfg-eth0文件，把文件中的ens32 都修改为 eth0

```shell
#编辑 ifcfg-eth0文件
vi ifcfg-eth0   
    
#把ifcfg-eth0文件中的 ens32 都修改为 eth0      
NAME=eth0      
DEVICE=eth0
```

![](https://www.codepeople.cn/imges/centos/install/045.png)

2、配置IP地址，网关

```powershell
cd /etc/sysconfig/network-scripts/    //进入到network-scripts目录下  
 
vi  ifcfg-eth0  //编辑配置文件 
 
#修改以下内容
BOOTPROTO=static  //启用静态IP地址
#DEVICE=eth0     //注释掉DEVICE 或者 删除DEVICE
ONBOOT=yes      //开启自动启用网络连接
 
 
//添加以下内容
IPADDR=192.168.0.179      //设置IP地址
NETMASK=255.255.255.0   //子网掩码
GATEWAY=192.168.0.2   //设置网关
```

![](https://www.codepeople.cn/imges/centos/install/046.png)

2.1、如何设置Linux的IP地址

​     1、在本地电脑打开“命令行窗口”，输入命令ipconfig，查看 以太网适配器 VMware Network Adapter VMnet8下的IPv4 地址

![](https://www.codepeople.cn/imges/centos/install/047.png)

2、可以看到 以太网适配器 VMware Network Adapter VMnet8下的IPv4 地址为 192.168.0.1

   3、当我们设置Linux的IP地址时，需要保证Linux的IP地址 与 VMnet8下的IPv4地址 的前面三位数必须相同，即 192.168.0 必须相同，这样Linux就能跟我们本地电脑互相通信，最后一位数不相同即可，例如Linux的IP地址可以设置为192.168.30.123 或者 192.168.30.124等

 

   2.2、如何设置Linux的网关：

​	1、点击编辑(E) → 虚拟网络编辑器(N)

![](https://www.codepeople.cn/imges/centos/install/048.png)

​        2、查看VMnet8 (NAT 模式)下的网关，选择VMnet8，再点击"NAT设置"按钮

![](https://www.codepeople.cn/imges/centos/install/049.png)

![](https://www.codepeople.cn/imges/centos/install/050.png)

 2.3、如何设置Linux的网关，查看VMnet8 (NAT 模式)下的子网掩码

![](https://www.codepeople.cn/imges/centos/install/051.png)

3、重启网络服务

```powershell
service network restart
```

![](https://www.codepeople.cn/imges/centos/install/052.png)

4、设置DNS地址 

```powershell
vi /etc/resolv.conf    //编辑 resolv.conf文件
 
nameserver 114.114.114.114   //添加DNS地址
 
 
 
常用的DNS地址:
 
  阿里  223.5.5.5  或者  223.6.6.6
 
  谷歌  8.8.8.8
  
  国内移动、电信和联通通用的DNS  114.114.114.114
```

![](https://www.codepeople.cn/imges/centos/install/053.png)

5、永久关闭 iptables（centos7默认的防火墙是firewalld防火墙，不是使用iptables，因此需要关闭firewalld服务）

```powershell
systemctl stop firewalld.service  // 停止firewalld服务
 
systemctl disable firewalld.service // 开机禁用firewalld服务
 
iptables -L   //列出所有iptables规则
```

![](https://www.codepeople.cn/imges/centos/install/054.png)

6、永久关闭SELinux防火墙

```powershell
vi /etc/sysconfig/selinux       //编辑selinux文件
 
SELINUX=disabled         //把文件中的SELINUX=enforcing 改成 SELINUX=disabled 即可
 
sestatus    //查看SELinux状态
```

![](https://www.codepeople.cn/imges/centos/install/055.png)

此时获取当前selinux防火墙的安全策略仍为enforcing，配置文件并未生效

![](https://www.codepeople.cn/imges/centos/install/056.png)

这时需要我们重启，再去查看SELinux防火墙的状态，可以看到已经关闭了

![](https://www.codepeople.cn/imges/centos/install/057.png)

7、给/etc/rc.d/rc.local 文件设置  “x”可执行权限，最初设置默认是没有可执行权限的

```powershell
chmod +x /etc/rc.d/rc.local     //设置可执行权限
 
 
//设置前
-rwxr-xr--. 1 root root 473 Aug  5  2017 rc.local
 
//设置后
-rwxr-xr-x. 1 root root 473 Aug  5  2017 rc.local
```

![](https://www.codepeople.cn/imges/centos/install/058.png)

8、输入“halt”，关闭虚拟机，并拍摄快照，保存当前配置

![](https://www.codepeople.cn/imges/centos/install/059.png)

![](https://www.codepeople.cn/imges/centos/install/060.png)



原文链接：<https://blog.csdn.net/qq_39135287/article/details/83993574>

=====================================================================

博客地址：<https://www.codepeople.cn>

=====================================================================

微信公众号：

![](https://www.codepeople.cn/imges/weixin_icon/weixin.jpg)