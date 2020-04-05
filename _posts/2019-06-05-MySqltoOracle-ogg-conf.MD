---
title:  "CentOS 下MySQL到Oracle同步的OGG解决方案"
categories: CentOS MySQL Oracle
tags: CentOS MySQL Oracle
author: LIUREN
---

# CentOS 下MySQL到Oracle同步的OGG解决方案

>  为了保证数据能够实时同步问题，公司提出使用ogg解决方案，这样能够把MySQL中的数据实时同步到Oracle中。两边保持同步。



Oracle Golden Gate（简称OGG）提供异构环境下交易数据的实时捕捉、变换、投递。

![](https://www.codepeople.cn/imges/ogg/001.png)

OGG支持的异构环境有：

![](https://www.codepeople.cn/imges/ogg/002.png)

OGG的工作原理：

![](https://www.codepeople.cn/imges/ogg/003.png)

OGG的进程：

![](https://www.codepeople.cn/imges/ogg/004.png)

- Manager进程是GoldenGate的控制进程，运行在源端和目标端上。它主要作用有以下几个方面：启动、监控、重启Goldengate的其他进程，报告错误及事件，分配数据存储空间，发布阀值报告等。在目标端和源端有且只有一个manager进程

- Extract运行在数据库源端，负责从源端数据表或者日志中捕获数据。Extract的作用可以按照阶段来划分为：

  - 初始时间装载阶段：在初始数据装载阶段，Extract进程直接从源端的数据表中抽取数据
  - 同步变化捕获阶段：初始数据同步完成以后，Extract进程负责捕获源端数据的变化(DML和DDL)

- Data Pump进程运行在数据库源端，其作用是将源端产生的本地trail文件，把trail以数据块的形式通过TCP/IP 协议发送到目标端，这通常也是推荐的方式。pump进程本质是extract进程的一种特殊形式，如果不使用trail文件，那么extract进程在抽取完数据以后，直接投递到目标端，生成远程trail文件。

- Collector进程与Data Pump进程对应 的叫Server Collector进程，这个进程不需要引起我的关注，因为在实际操作过程中，无需我们对其进行任何配置，所以对我们来说它是透明的。它运行在目标端，其 任务就是把Extract/Pump投递过来的数据重新组装成远程ttrail文件。

- Replicat进程，通常我们也把它叫做应用进程。运行在目标端，是数据传递的最后一站，负责读取目标端trail文件中的内容，并将其解析为DML或 DDL语句，然后应用到目标数据库中。

  **关于OGG的Trail文件：**

- 为了更有效、更安全的把数据库事务信息从源端投递到目标端。GoldenGate引进trail文件的概念。前面提到extract抽取完数据以后 Goldengate会将抽取的事务信息转化为一种GoldenGate专有格式的文件。然后pump负责把源端的trail文件投递到目标端，所以源、目标两端都会存在这种文件。

- trail文件存在的目的旨在防止单点故障，将事务信息持久化，并且使用checkpoint机制来记录其读写位置，如果故障发生，则数据可以根据checkpoint记录的位置来重传 。



**1、应用场景**

1)高可用容灾

2)数据库迁移、升级（支持跨版本、异构数据库、零宕机时间、亚秒级恢复）

3)实时数据集成（支持异构数据库、多源数据库）

**2、常用拓扑**

（下图来自网络）

![](https://www.codepeople.cn/imges/ogg/005.png)



**3、支持的平台和数据库**

目前暂时知道是 Oracle、MySQL、SQLserver、Postgresql

做过实验的就只有：单向复制：MySQL===>Oracle

## 第一步： 前提条件准备

我的环境介绍

| 实验环境     | 源端软件版本                                                 | 目标软件版本                                                 |
| ------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 操作系统     | CentOS7.4 64位                                               | CentOS7.4 64位                                               |
| IP地址       | 192.168.0.137                                                | 192.168.0.223                                                |
| 数据库       | mysql-8.0.11                                                 | oracle-11.2                                                  |
| 数据账号密码 | root/root                                                    | root/root                                                    |
| 数据库       | tb_user;tb_log;tb_role                                       | tb_user;tb_log;tb_role                                       |
| GoldenGate   | [Oracle GoldenGate 19.1.0.0.0 for MySQL on Linux x86-64  ](https://www.oracle.com/technetwork/middleware/goldengate/downloads/index.html)(67 MB) | [Oracle GoldenGate 19.1.0.0.1 for Oracle on Linux x86-64  ](https://www.oracle.com/technetwork/middleware/goldengate/downloads/index.html)(520 MB) |

## 第二步：确保MySQL服务和Oracle服务都能正常启动

MySQL环境的安装，Oracle环境的安装过程需要自己去实现

可以参考：<https://blog.csdn.net/qq_32786873/article/details/82110235>

MySQL的CentOS安装网上比较多可以百度



## 第三步: 修改MySQL的配置文件my.cnf

`vim my.cnf` 或者 `vi my.cnf`

```
server-id=1
log-bin=mysql-bin
binlog-format=row
```



## 第四步：下载安装对应的ogg版本

链接: https://pan.baidu.com/s/1Rc6zUT1yJGR1k7P23Oowjw 提取码: xkza 



## 第五步：开始MYSQL源端ogg配置

1.首先删除文件目录（如果存在）

```
rm -rf dirprm dirrpt dirchk dirpcs dirsql dirdef dirdat dirtmp dircrd dirwlt dirdmp
```

注释： 删除的这些目录第一次是没有的，不需要执行这个命令

2.置空日志信息

```
echo '' > /home/ucenter/soft/ggs/ggserr.log
```

注释：一般不建议置空日志，不利于检查错误日志，如果都配置正常跑起也正常后可以清空以前部署日志信息

3.进入ogg命令行界面

```
./ggsci
GGSCI (ogg) 1> create subdirs
Creating subdirectories under current directory /u01/ogg_ms
Parameter files                /u01/ogg_ms/dirprm: already exists
Report files                   /u01/ogg_ms/dirrpt: created
Checkpoint files               /u01/ogg_ms/dirchk: created
Process status files           /u01/ogg_ms/dirpcs: created
SQL script files               /u01/ogg_ms/dirsql: created
Database definitions files     /u01/ogg_ms/dirdef: created
Extract data files             /u01/ogg_ms/dirdat: created
Temporary files                /u01/ogg_ms/dirtmp: created
Stdout files                   /u01/ogg_ms/dirout: created
```

注释：集体创建目录

4.ogg命令行登录数据库

```
dblogin sourcedb kelan@localhost:3306,userid root,password root
```

注释：kelan这个是数据库库名称 userid 是指登录账号 password 是指登录密码

-- 开始配置MYSQL端的mgr

```
edit params mgr
```

```
port 7809
dynamicportlist 7800-8000
autorestart extract *,waitminutes 2,resetminutes 5
```

```
info mgr
start mgr
info mgr
```

-- 配置抽取进程

```
edit params yhklext
```

```
extract yhklext
setenv (MYSQL_HOME="/var/lib/mysql")
tranlogoptions altlogdest /var/lib/mysql/mysql-bin.index
sourcedb bsdt@88.20.16.100:3306, userid root,password 123456
exttrail ./dirdat/kl,FORMAT RELEASE 12.2
gettruncates
TABLE bsdt.bsdt_ckxx, KEYCOLS(CKXH);
TABLE bsdt.bsdt_fwmx, KEYCOLS(FWMXXH);
TABLE bsdt.bsdt_jbxx, KEYCOLS(BSFWT_DM);
TABLE bsdt.bsdt_jkxx, KEYCOLS(JKXXXH);
TABLE bsdt.bsdt_phmx, KEYCOLS(PHMXXH);
TABLE bsdt.bsdt_ryzxqk, KEYCOLS(BSFWT_DM);
TABLE bsdt.bsdt_ywmx, KEYCOLS(YWMXXH);
TABLE bsdt.bsdt_yymx, KEYCOLS(YYMXXH);
TABLE bsdt.bsdt_yyrsxz, KEYCOLS(BSFWT_DM);
```

注释：MYSQL_HOME是指你自己的MYSQL安装目录， FORMAT RELEASE 12.2是指支持的ogg版本是12.2可以删除



-- 配置传输进程

```
edit params yhklpump
```

```
extract yhklpump
rmthost 88.16.41.33,mgrport 7809
rmttrail ./dirdat/kl
passthru
gettruncates
TABLE bsdt.bsdt_ckxx, KEYCOLS(CKXH);
TABLE bsdt.bsdt_fwmx, KEYCOLS(FWMXXH);
TABLE bsdt.bsdt_jbxx, KEYCOLS(BSFWT_DM);
TABLE bsdt.bsdt_jkxx, KEYCOLS(JKXXXH);
TABLE bsdt.bsdt_phmx, KEYCOLS(PHMXXH);
TABLE bsdt.bsdt_ryzxqk, KEYCOLS(BSFWT_DM);
TABLE bsdt.bsdt_ywmx, KEYCOLS(YWMXXH);
TABLE bsdt.bsdt_yymx, KEYCOLS(YYMXXH);
TABLE bsdt.bsdt_yyrsxz, KEYCOLS(BSFWT_DM);
```

注释：rmthost 是指传送的目的端，端口号是 7809 端口是固定值，记得CentOS下要放开此端口

-- 初始化传输工具

```
edit params yhklinit
```

```
EXTRACT yhklinit
SETENV (NLS_LANG=AMERICAN_AMERICA.AL32UTF8)
sourcedb bsdt@88.20.16.100:3306, userid root,password 123456
rmthost 88.16.41.33,mgrport 7809
RMTTASK REPLICAT, GROUP yhklinit
TABLE bsdt.bsdt_ckxx, KEYCOLS(CKXH);
TABLE bsdt.bsdt_fwmx, KEYCOLS(FWMXXH);
TABLE bsdt.bsdt_jbxx, KEYCOLS(BSFWT_DM);
TABLE bsdt.bsdt_jkxx, KEYCOLS(JKXXXH);
TABLE bsdt.bsdt_phmx, KEYCOLS(PHMXXH);
TABLE bsdt.bsdt_ryzxqk, KEYCOLS(BSFWT_DM);
TABLE bsdt.bsdt_ywmx, KEYCOLS(YWMXXH);
TABLE bsdt.bsdt_yymx, KEYCOLS(YYMXXH);
TABLE bsdt.bsdt_yyrsxz, KEYCOLS(BSFWT_DM);
```

-- 异构平台配置defgen

```
edit params defgen
```

```
defsfile ./dirdef/defgen.prm
sourcedb bsdt@88.20.16.100:3306, userid root,password 123456
TABLE bsdt.bsdt_ckxx;
TABLE bsdt.bsdt_fwmx;
TABLE bsdt.bsdt_jbxx;
TABLE bsdt.bsdt_jkxx;
TABLE bsdt.bsdt_phmx;
TABLE bsdt.bsdt_ryzxqk;
TABLE bsdt.bsdt_ywmx;
TABLE bsdt.bsdt_yymx;
TABLE bsdt.bsdt_yyrsxz;
```

--执行copy命令、在ogg的安装目录下

```
[ucenter]$./defgen paramfile dirprm/defgen.prm
[ucenter]$ scp dirdef/defgen.prm tmp01@88.16.41.33:/tmp01
```

--源端执行命令,添加进程组

```
add extract yhklext,tranlog,begin now
add exttrail ./dirdat/kl,extract yhklext,MEGABYTES 500
add extract yhklpump,exttrailsource ./dirdat/kl
add rmttrail ./dirdat/kl,extract yhklpump, MEGABYTES 500
add extract yhklinit,sourceistable
```

-- 执行启动命令==最好是在MySQL端启动后，Oracle端也马上执行启动命令==

```
start yhklinit
start extract yhklext
start extract yhklpump

```

## 第六步：配置Oracle端

1.首先删除文件目录（如果存在）

```
rm -rf dirprm dirrpt dirchk dirpcs dirsql dirdef dirdat dirtmp dircrd dirwlt dirdmp
```

2.置空日志信息

```
echo '' > /home/ucenter/soft/ggs/ggserr.log
```

3.进入ogg命令行界面

```
./ggsci
GGSCI (ogg) 1> create subdirs
Creating subdirectories under current directory /u01/ogg_ms
Parameter files                /u01/ogg_ms/dirprm: already exists
Report files                   /u01/ogg_ms/dirrpt: created
Checkpoint files               /u01/ogg_ms/dirchk: created
Process status files           /u01/ogg_ms/dirpcs: created
SQL script files               /u01/ogg_ms/dirsql: created
Database definitions files     /u01/ogg_ms/dirdef: created
Extract data files             /u01/ogg_ms/dirdat: created
Temporary files                /u01/ogg_ms/dirtmp: created
Stdout files                   /u01/ogg_ms/dirout: created
```

4.ogg命令行登录数据库

```
dblogin userid root,password root
```

注释：跟MySQL端的登录方式不一样，这个是登录Oracle数据库

--配置mgr

```
edit params mgr
```

```
port 7809
dynamicportlist 7800-8000
autorestart extract *,waitminutes 2,resetminutes 5
accessrule,prog *,ipaddr 192.168.*,allow
```

```
info mgr
start mgr
info mgr
```

注意：`accessrule,prog *,ipaddr 192.168.*,allow` 意思是允许 IP为192.168.*的进行数据传送

--配置复制进程

```
edit params yhklrepl
```

```
replicat yhklrepl
sourcedefs /u01/app/ogginstall/dirdef/defgen.prm
userid root,password root
REPERROR DEFAULT, DISCARD
discardfile ./dirrpt/yh_yhkl.dsc,append,megabytes 50
dynamicresolution
map bsdt.bsdt_ckxx, target bsdt.bsdt_ckxx,KEYCOLS(CKXH);
map bsdt.bsdt_fwmx, target bsdt.bsdt_fwmx,KEYCOLS(FWMXXH);
map bsdt.bsdt_jbxx, target bsdt.bsdt_jbxx,KEYCOLS(BSFWT_DM);
map bsdt.bsdt_jkxx, target bsdt.bsdt_jkxx,KEYCOLS(JKXXXH);
map bsdt.bsdt_phmx, target bsdt.bsdt_phmx,KEYCOLS(PHMXXH);
map bsdt.bsdt_ryzxqk, target bsdt.bsdt_ryzxqk,KEYCOLS(BSFWT_DM);
map bsdt.bsdt_ywmx, target bsdt.bsdt_ywmx,KEYCOLS(YWMXXH);
map bsdt.bsdt_yymx, target bsdt.bsdt_yymx,KEYCOLS(YYMXXH);
map bsdt.bsdt_yyrsxz, target bsdt.bsdt_yyrsxz,KEYCOLS(BSFWT_DM);
```

--配置初始化接收进程

```
edit params yhklinit
```

```
replicat yhklinit
--SETENV (NLS_LANG=AMERICAN_AMERICA.AL32UTF8)
userid root,password root
sourcedefs ./dirdef/defgen.prm
--HANDLECOLLISIONS
REPERROR (-1,IGNORE)
REPERROR DEFAULT, DISCARD
map bsdt.bsdt_ckxx, target bsdt.bsdt_ckxx,KEYCOLS(CKXH);
map bsdt.bsdt_fwmx, target bsdt.bsdt_fwmx,KEYCOLS(FWMXXH);
map bsdt.bsdt_jbxx, target bsdt.bsdt_jbxx,KEYCOLS(BSFWT_DM);
map bsdt.bsdt_jkxx, target bsdt.bsdt_jkxx,KEYCOLS(JKXXXH);
map bsdt.bsdt_phmx, target bsdt.bsdt_phmx,KEYCOLS(PHMXXH);
map bsdt.bsdt_ryzxqk, target bsdt.bsdt_ryzxqk,KEYCOLS(BSFWT_DM);
map bsdt.bsdt_ywmx, target bsdt.bsdt_ywmx,KEYCOLS(YWMXXH);
map bsdt.bsdt_yymx, target bsdt.bsdt_yymx,KEYCOLS(YYMXXH);
map bsdt.bsdt_yyrsxz, target bsdt.bsdt_yyrsxz,KEYCOLS(BSFWT_DM);
```

--目的端执行命令

```
add replicat yhklrepl,exttrail ./dirdat/kl,checkpointtable root.checkpointtab
add replicat yhklinit,specialrun
```

-- 执行启动命令==最好是在MySQL端启动后，Oracle端也马上执行启动命令==

```
start replicat yhklrepl
start yhklinit
```



## 第七步：复查配置

--主要复查的是命令是

```
scp dirdef/defgen.prm oracle@192.168.0.223:/u01/app/ogginstall/dirdef
```

查看copy到Oracle中的文件是否正确，dirdef中的文件内容如下：

```
*+- Defgen version 7.0, Encoding UTF-8
*
* Definitions created/modified  2019-06-05 14:31
*
*  Field descriptions for each column entry:
*
*     1    Name
*     2    Data Type
*     3    External Length
*     4    Fetch Offset
*     5    Scale
*     6    Level
*     7    Null
*     8    Bump if Odd
*     9    Internal Length
*    10    Binary Length
*    11    Table Length
*    12    Most Significant DT
*    13    Least Significant DT
*    14    High Precision
*    15    Low Precision
*    16    Elementary Item
*    17    Occurs
*    18    Key Column
*    19    Sub Data Type
*    20    Native Data Type
*    21    Character Set
*    22    Character Length
*    23    LOB Type
*    24    Partial Type
*
Database type: MYSQL
Character set ID: UTF-8
National character set ID: UTF-16BE
Locale: en_US
Case sensitivity: 11 11 11 22 22 11 11 11 11 11 11 11 11 44 11 11
TimeZone: Asia/Shanghai
*
Definition for table kelan.tb_user
Record length: 2057
Syskey: 0
Columns: 3
ID        132     11        0  0  0 1 0      4      4      4 0 0 0 0 1    0 1   0    3       -1      0 0 0
USERNAME   64   1020        7  0  0 1 0   1020   1020      0 0 0 0 0 1    0 0   4  253        0      0 0 0
PASSWORD   64   1020     1032  0  0 1 0   1020   1020      0 0 0 0 0 1    0 0   4  253        0      0 0 0
End of definition
*
Definition for table kelan.tb_role
Record length: 2057
Syskey: 0
Columns: 3
ID        132     11        0  0  0 1 0      4      4      4 0 0 0 0 1    0 1   0    3       -1      0 0 0
ROLENAME   64   1020        7  0  0 1 0   1020   1020      0 0 0 0 0 1    0 0   4  253        0      0 0 0
ROLENUM    64   1020     1032  0  0 1 0   1020   1020      0 0 0 0 0 1    0 0   4  253        0      0 0 0
End of definition
*
Definition for table kelan.tb_log
Record length: 2057
Syskey: 0
Columns: 3
ID       132     11        0  0  0 1 0      4      4      4 0 0 0 0 1    0 1   0    3       -1      0 0 0
LOGNAME   64   1020        7  0  0 1 0   1020   1020      0 0 0 0 0 1    0 0   4  253        0      0 0 0
LOGNUM    64   1020     1032  0  0 1 0   1020   1020      0 0 0 0 0 1    0 0   4  253        0      0 0 0
End of definition
```

==如果内容不是类似的话就是copy错误，需要重新copy一次==



## 第八步：中间遇到的问题

遇到问题1
```
Failed to read SOURCEDEFS file /u01/app/ogginstall/dirdef/defgen.prm: Could not read /u01/app/ogginstall/dirdef/defgen.prm: No such file or directory.
```

解决方案：该问题不是系统读取不到文件，是因为从源端copy到目的端的文件错误，删除重新copy

```
rm -rf /u01/app/ogginstall/dirdef/defgen.prm
## 然后到mysql的源端重新执行scp命令
scp dirdef/defgen.prm oracle@192.168.0.223:/u01/app/ogginstall/dirdef
```

遇到问题2

```
Checkpoint table root.CKPTTABLE does not exist. Create this table with the ADD CHECKPOINTTABLE command.
```

解决方案：该问题是数据库表中找不到CHECKPOINTTABLE表,需要手动创建

​		    到Oracle的的登录界面，Oracle的登录方法 sqlplus root/root

```
SQL> sqlplus root/root
SQL> select count(*) from dba_tables where owner='ROOT';
SQL> select table_name from dba_tables where owner='ROOT';
```

查询结果如下：

```
--------------------------------------------------------------------------------
CHECKPOINTTAB_LOX
CHECKPOINTTAB
```

删除这两张表

```
SQL> drop table CHECKPOINTTAB_LOX;
SQL> drop table CHECKPOINTTAB;
```

然后重新执行ogg的  命令重新建表

```
./ggsci
dblogin userid root ,password root
add checkpointtable root.checkpointtab
```

遇到问题3

```
Confirm that the log file exists, that the path is correct, and that the correct permissions are set for Oracle GoldenGate
Also try specifying the path to the log index file by using the TRANLOGOPTIONS parameter with the ALTLOGDEST option. - /var/lib/mysql/mysql-bin.index
                              WHEN FAILED : While initializing binary log configuration
                              WHERE FAILED : MySQLBinLog Reader Module
                              CONTEXT OF FAILURE : No Information Available!>.
```

解决方案：`cd /var/lib/mysql/` 目录下，执行 `sudo chmod 777 -R *`

正常执行后MySQL端的的效果

```
GGSCI (ogg) 7> info all
Program     Status      Group       Lag at Chkpt  Time Since Chkpt
MANAGER     RUNNING
EXTRACT     RUNNING     YHXXEXT       00:00:00      00:21:17
EXTRACT     RUNNING     YHXXPUMP      00:00:00      00:00:09
```

正常执行Oracle端的的效果

```
GGSCI (localhost.localdomain as root@orcl) 5> info all

Program     Status      Group       Lag at Chkpt  Time Since Chkpt

MANAGER     RUNNING                                           
REPLICAT    RUNNING     YHXXREPL    00:00:00      00:00:09  
```

==yhxxinit进程是看不到的==

第九步：日常使用命令和方法

查看ogg启动报错信息的两种方法

```
$ ./ggsci
GGSCI (ogg) 1> view ggsevt
```

或者

```
$ cd /home/ucenter/soft/ggs
$ tail -f ggserr.log
```



## 最后一步： 确认CentOS的端口是开放的，如果不是开放的可能会有问题

```
需要开放的端口如下
#MySQL的CentOS端
3306  7809
#Oracle的CentOS端
3306 7809 1521
```

端口开发问题没有测试过，为了测试方便我直接把防火墙关闭了

一、防火墙的开启、关闭、禁用命令

```
(1）设置开机启用防火墙：systemctl enable firewalld.service

（2）设置开机禁用防火墙：systemctl disable firewalld.service

（3）启动防火墙：systemctl start firewalld

（4）关闭防火墙：systemctl stop firewalld

（5）检查防火墙状态：systemctl status firewalld 
```

二、使用firewall-cmd配置端口

（1）查看防火墙状态：firewall-cmd --state

（2）重新加载配置：firewall-cmd --reload

（3）查看开放的端口：firewall-cmd --list-ports

（4）开启防火墙端口：firewall-cmd --zone=public --add-port=9200/tcp --permanent

　　命令含义：

　　–zone #作用域

　　–add-port=9200/tcp #添加端口，格式为：端口/通讯协议

　　–permanent #永久生效，没有此参数重启后失效

　　**注意：添加端口后，必须用命令firewall-cmd --reload重新加载一遍才会生效**

（5）关闭防火墙端口：firewall-cmd --zone=public --remove-port=9200/tcp --permanent





以上内容是真实环境测试过的。数据能够正常同步。端口问题有时间再进行测试

================================================================

[博客地址](https://www.codepeople.cn)：<https://www.codepeople.cn>

==================================================================

微信公众号：

![](https://www.codepeople.cn/imges/weixin_icon/weixin.jpg)