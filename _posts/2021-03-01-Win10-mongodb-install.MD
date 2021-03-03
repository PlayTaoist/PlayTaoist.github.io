---
title:  "Win10 系统下安装绿色版mongodb服务"
categories: Win10 mongodb
tags: Win10 mongodb
author: LIUREN
---

# Win10 系统下安装绿色版mongodb服务

> MongoDB 是一个基于分布式文件存储的数据库。由 C++ 语言编写。旨在为 WEB 应用提供可扩展的高性能数据存储解决方案。
>
> MongoDB 是一个介于关系数据库和非关系数据库之间的产品，是非关系数据库当中功能最丰富，最像关系数据库的。
>
> 协议：CC BY-SA 4.0 https://creativecommons.org/licenses/by-sa/4.0/  
>
> 版权声明：本文为原创文章，遵循 CC 4.0 BY-SA 版权协议，转载请附上原文出处链接及本声明。



## 1.MongoDB下载

MongoDB官网下载链接：https://www.mongodb.com/download-center/community
MongoDB官网安装教程：https://docs.mongodb.com/manual/tutorial/install-mongodb-on-windows/

MongoDB 4.4.4 版本下载地址

![](https://www.codepeople.cn/imges/00x82.png)



【绿色版.zip】https://fastdl.mongodb.org/windows/mongodb-windows-x86_64-4.4.4.zip

【安装版.msi】https://fastdl.mongodb.org/windows/mongodb-windows-x86_64-4.4.4-signed.msi



## 二.自定义安装目录

1.解压到：D:\mongodb\mongodb-win32-x86_64-windows-4.4.4

![](https://www.codepeople.cn/imges/00x76.png)



2.配置系统环境变量

![](https://www.codepeople.cn/imges/00x77.png)

![](https://www.codepeople.cn/imges/00x78.png)



3. 在安装路径下创建文件夹和文件：

"/data" 和 "/log" 两个文件夹（"/data"文件夹下，用于存放数据库源数据；"/log"文件夹下，用于存放logger日志）；
在data文件夹下创建"/db"空文件夹；
在log文件夹下创建"mongo.log"空文件。
注：其实data和log两个文件夹，在何处创建都无所谓，依个人习惯。关键在于下一步骤，"mongo.config"配置内容中，"dbpath"和"logpath"所指向的路径。

为将重要文件与非重要文件分离，以便数据移植，故data文件夹与log文件夹分开创建。

![](https://www.codepeople.cn/imges/00x79.png)



4. 在安装路径下创建配置文件“mongo.config”，并写入以下内容：

```properties
#数据库文件的存放位置
dbpath=D:\mongodb\mongodb-win32-x86_64-windows-4.4.4\data\db

#日志文件存放的路径
logpath=D:\mongodb\mongodb-win32-x86_64-windows-4.4.4\log\mongo.log
```



5. 在"D:\mongodb\mongodb-win32-x86_64-windows-4.4.4\bin"目录下运行cmd

![](https://www.codepeople.cn/imges/00x80.png)

运行完毕后出现如下界面

![](https://www.codepeople.cn/imges/00x81.png)



6. 输入命令

```cmd
mongod --config "D:\mongodb\mongodb-win32-x86_64-windows-4.4.4\mongo.config" --install --serviceName "MongoDB"
```



7. 启动MongoDB服务命令

```powershell
#启动服务
net start MongoDB
#停止服务
net stop MongoDB
```

8. 服务启动后如果想通过cmd界面操作链接

启动服务完成后，win + R直接运行"cmd"，输入"mongo"即可运行

```
C:\Users\PC>mongo
MongoDB shell version v4.4.4
connecting to: mongodb://127.0.0.1:27017
MongoDB server version: 4.4.14
Server has startup warnings:
2021-03-03T15:36:39.950+0800 I CONTROL  [initandlisten]
2021-03-03T15:36:39.950+0800 I CONTROL  [initandlisten] ** WARNING: Access control is not enabled for the database.
2021-03-03T15:36:39.950+0800 I CONTROL  [initandlisten] **          Read and write access to data and configuration is unrestricted.
2021-03-03T15:36:39.950+0800 I CONTROL  [initandlisten]
2021-03-03T15:36:39.950+0800 I CONTROL  [initandlisten] ** WARNING: This server is bound to localhost.
2021-03-03T15:36:39.950+0800 I CONTROL  [initandlisten] **          Remote systems will be unable to connect to this server.
2021-03-03T15:36:39.950+0800 I CONTROL  [initandlisten] **          Start the server with --bind_ip <address> to specify which IP
2021-03-03T15:36:39.950+0800 I CONTROL  [initandlisten] **          addresses it should serve responses from, or with --bind_ip_all to
2021-03-03T15:36:39.950+0800 I CONTROL  [initandlisten] **          bind to all interfaces. If this behavior is desired, start the
2021-03-03T15:36:39.950+0800 I CONTROL  [initandlisten] **          server with --bind_ip 127.0.0.1 to disable this warning.
2021-03-03T15:36:39.950+0800 I CONTROL  [initandlisten]
```



启动失败的话，需要先删除安装的服务，重新安装

服务的删除：

有些小伙伴通过 cmd->services.msc 找到 mongodb服务，但却无法删除

方法：通过cmd 进入安装的mongodb路径，进入bin目录，使用命令 mongod.exe --remove --serviceName "MongoDB"

使用Mongodb做一个评论功能例子：https://www.cnblogs.com/lr393993507/p/14470525.html



------

=====================================================================

微信公众号：

![](https://www.codepeople.cn/imges/weixin_icon/weixin.jpg)