---
title:  "工作中要使用的git命令记录下来"
categories:  Git
tags: Git
author: LIUREN
---

# 工作中要使用的git命令记录下来

> 主要是为了记录工作中常用的Git命令，这个会慢慢地更新记录，主要是为了让自己更好的记住和方便查阅；
>
> 协议：CC BY-SA 4.0 https://creativecommons.org/licenses/by-sa/4.0/  
>
> 版权声明：本文为原创文章，遵循 CC 4.0 BY-SA 版权协议，转载请附上原文出处链接及本声明。



[TOC]

## Git修改分支

#### 第一步：进入项目中（进入项目文件夹里，然后打开Git命令行）

```shell
git branch -a ##这个主要是查询有多少个分支信息
git checkout develop ##该命令是切换到develop分支下
git pull ##切换到分支下后，再更新一下文件信息
```



## Git修改提交的账号和邮箱

有时候我们已经换了账号和邮箱信息，需要切换到更符合要求的账号和邮箱信息；我们直接在桌面上右键打卡git命令行

```shell
git config --list ### 查询所有的Git配置信息
git config --global --replace-all user.name "用户名"     	###Git上显示的提交代码的名称
git config --global --replace-all user.email "用户邮箱"   
```



其余的用上了再慢慢记录!



=====================================================================

微信公众号：

![](https://www.codepeople.cn/imges/weixin_icon/weixin.jpg)