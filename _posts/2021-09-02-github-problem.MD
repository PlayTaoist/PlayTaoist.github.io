---
title:  "【突发】解决remote: Support for password authentication was removed on August 13, 2021. Please use a perso"
categories:  Git
tags: Git
author: LIUREN
---

# 【突发】解决remote: Support for password authentication was removed on August 13, 2021. Please use a perso

> 【突发】解决remote: Support for password authentication was removed on August 13, 2021. Please use a perso；**Github不能拉代码了？**
>
> 协议：CC BY-SA 4.0 https://creativecommons.org/licenses/by-sa/4.0/  
>
> 版权声明：本文为原创文章，遵循 CC 4.0 BY-SA 版权协议，转载请附上原文出处链接及本声明。



[TOC]

## 今天我像往常那样提交一下Github代码，发现竟然报了下面的错误：

#### 第一步：进入项目中（进入项目文件夹里，然后打开Git命令行）

```shell
git push
remote: Support for password authentication was removed on August 13, 2021. Please use a personal access token instead.
remote: Please see https://github.blog/2020-12-15-token-authentication-requirements-for-git-operations/ for more information.
fatal: Authentication failed for 'https://github.com/PlayTaoist/PlayTaoist.github.io.git/'
```

从2021年8月13后不再支持用户名密码的方式验证了，需要创建个人访问令牌(personal access token)。



## 解决方法

首先[生成一个令牌](https://docs.github.com/en/github/authenticating-to-github/keeping-your-account-and-data-secure/creating-a-personal-access-token)，←链接是官网教程，也可以不点，直接看下文。

1. 点击Settings

![](https://www.codepeople.cn/imges/00x118.png)

2.点击左侧的Developer settings
![](https://www.codepeople.cn/imges/00x119.png)

3.点击Personal access tokens(个人访问令牌)
![](https://www.codepeople.cn/imges/00x120.png)

4.点击Generate new token
![](https://www.codepeople.cn/imges/00x121.png)

5.设置token信息
![](https://www.codepeople.cn/imges/00x122.png)
点击Generate token生成令牌

6.得到生成的令牌
![](https://www.codepeople.cn/imges/00x123.png)

将生成的令牌拷贝下来(不要拷贝我的，拷贝你自己生成的!)，记得保存，下次你就看不到了。

修改现有项目的url

```shell
git remote set-url origin  https://<your_token>@github.com/<USERNAME>/<REPO>.git
```
将<your_token>换成你自己得到的令牌。<USERNAME>是你自己github的用户名，<REPO>是你的项目名称，比如我的：

```shell
git remote set-url origin https://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@github.com/PlayTaoist/PlayTaoist.github.io.git
```

最后执行 git push;
```shell
git push
```
就可以正常提交了



=====================================================================

微信公众号：

![](https://www.codepeople.cn/imges/weixin_icon/weixin.jpg)