---
title:  "Windows下Kettle定时任务执行并发送错误信息邮件"
categories: Kettle
tags: Kettle
author: LIUREN
---

## Windows下Kettle定时任务执行并发送错误信息邮件

>Windows下Kettle定时任务执行并发送错误信息邮件,使用Kettle可以同步数据库信息，也可以直接集成到Java项目中用于项目开发工作



## 1.首先安装JDK

#### 2.配置JDK环境

#### 3.下载并解压PDI(kettle)

目前我用的是版本V7的，可以直接百度搜索下载社区版，企业版收费，社区版免费

下载地址：https://sourceforge.net/projects/pentaho/files/Data%20Integration/

![018](https://www.codepeople.cn/imges/kettle_png/018.png)

#### 4.打开解压文件夹kettle找到spoon.bat，双击打开

#### 5.新建作业

点击左上角`文件`=>`新建`=>`作业`

如下图：

![001](https://www.codepeople.cn/imges/kettle_png/001.png)

#### 6.然后开始建立作业

流程如下图，直接拉图标就行

![002](https://www.codepeople.cn/imges/kettle_png/002.png)

#### 7.编辑 “转换”

右键编辑转换选择`Open Referenced Object `、`Transformation`然后新建一个转换

![003](https://www.codepeople.cn/imges/kettle_png//003.png)

#### 8.新建的转换流程和配置如下

`我做的是表的数据同步到另一个表中，只是做了简单的处理，没有做冲突验证`

![004](https://www.codepeople.cn/imges/kettle_png/004.png)

![005](https://www.codepeople.cn/imges/kettle_png/005.png)

![006](https://www.codepeople.cn/imges/kettle_png/006.png)



然后选择点击`获取SQL查询语句`

可以预览查看数据

#### 9.输出表配置

配置输出表的数据库连接新建数据库连接然后选择目标表

![/imges/kettle_png/007](https://www.codepeople.cn/imges/kettle_png/007.png)

其余的三个都是不同的表而已

#### 10.设置转换日志

双击转换然后设置日志信息



![008](https://www.codepeople.cn/imges/kettle_png/008.png)

其他的转换也同样设置，有几个转换就设置几个错误日志发送邮箱。`日志级别最好选择详情日志`

#### 11.邮箱设置

![009](https://www.codepeople.cn/imges/kettle_png/009.png)

![010](https://www.codepeople.cn/imges/kettle_png/010.png)

![011](https://www.codepeople.cn/imges/kettle_png/011.png)

![012](https://www.codepeople.cn/imges/kettle_png/012.png)



**其他邮箱也要这么设置。**

#### 12.编写完毕后就可以直接运行

**点击右上角的运行符号**跑起来后，不管成功或者失败都会发送邮件到你的邮箱。然后登录自己的邮箱就可以从邮件中直接查看错误日志了。附件中有详细的错误日志信息

![013](https://www.codepeople.cn/imges/kettle_png/013.png)

#### 13.怎么在windows中设置定时任务

`当然可以直接在START中设置定时任务的` 不过不如做成脚本直接让windows帮忙运行

保存作业生成一个后缀为kjb的一个文件，然后编写一个bat脚本文件：

```
@echo "该脚本这么解释，先进入kitchen.bat所在目录，执行kitchen 后面为所带参数"
cd /d D:
cd D:\Kettle
kitchen /file E:\kettle_files\timing.kjb /level Basic /logfile E:\kettle_logs\timing.log
@pause
```

cd /d D:  意思是强制进入D盘

cd D:\Kettle 意思是进入到Kettle的kitchen.bat所在目录中

kitchen /file E:\kettle_files\timing.kjb /level Basic /logfile E:\kettle_logs\timing.log  是使用kitchen的方式执行timing.kjb的作业，并在E盘的kettle_logs下生成执行的日志



#### 14.返回桌面，右键此电脑，进入管理，右键-任务计划程序-新建基本任务

定时任务参考地址：https://blog.csdn.net/huiweizuotiandeni/article/details/54382639



![014](https://www.codepeople.cn/imges/kettle_png/014.png)

![015](https://www.codepeople.cn/imges/kettle_png/015.png)

![016](https://www.codepeople.cn/imges/kettle_png/016.png)

![017](https://www.codepeople.cn/imges/kettle_png/017.png)