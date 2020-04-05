---
title:  "SpringBoot2.x开源项目和组建发现"
categories: SpringBoot2.0
tags: SpringBoot2.0
author: LIUREN
---

# SpringBoot2.x开源项目和组建发现

> 在学习过程中发现了不少有用的开源的基础框架，也找到了不少组建



## 安装环境

```shell
开发工具：STS
Maven版本：apache-maven-3.5.2
java jdk 1.8
MySQL版本：5.7
```



## 一.开源项目

> 猫宁Morning公益商城是中国公益性在线电子商城，以商城B2C模式运营的公益在线商城，是一家致力于将传统公益商城互联网化的创新公益商城。系统分为前台和后台,前台主要功能包括注册和登录、商品浏览、个人信息管理、购物车、我的订单、网上支付、评价、提问、公益活动、公益论坛、商城论坛等系统，后台主要功能包括管理成员信息管理、用户信息管理、商品管理、公益活动管理、商品分类管理、订单管理、公告资讯管理、论坛平台管理、商品提上管理、商品评价管理等。

[访问地址：](https://gitee.com/VCS/Morning)<https://gitee.com/VCS/Morning>



> **youngcms**基于SSM框架的内容管理系统，采用最新最主流的技术，后端采用spring boot,mybatis-plus,freemaker，shiro，redis，mysql，DWZ等技术,主要功能：消息队列，权限控制，自定义工作流，扩展模型，内容管理，通用日志记录等

[访问地址：](https://gitee.com/VCS/youngcms)<https://gitee.com/VCS/youngcms>



> MSFM权限管理系统是基于springBoot开发的一套轻量级的系统脚手架，可以以此形成一套属于自己的系统后台 ，自动生成前后台基本代码；使用Spring Boot、Shiro、MyBatis、Layui等框架，包含：用户管理、角色管理、权限管理、资源管理、数据库管理、源码生成、调度管理等

[访问地址：](https://gitee.com/VCS/mysiteforme)<https://gitee.com/VCS/mysiteforme>



>- guns-lite是在[guns](https://github.com/stylefeng/Guns)的基础上将数据库层由mybatis替换为spring data jpa的系统。
>
>- guns-lite是一个基于spring boot的后台管理系统。

[访问地址：](https://gitee.com/VCS/guns-lite)<https://gitee.com/VCS/guns-lite>



> [微同商城](https://gitee.com/fuyang_lipengjun/platform) 减少重复造轮子，开发微信小程序商城 (后台开源) 。快速搭建一个属于自己的微信小程序商城。QQ交流群：66502035、870579539欢迎大家进群交流技术。

[访问地址:](https://gitee.com/VCS/platform)<https://gitee.com/VCS/platform>



> *renren-security*采用SpringBoot2.0、MyBatis、Shiro框架，开发的一套权限系统，极低门槛，拿来即用。设计之初，就非常注重安全性，为企业系统保驾护航，让一切都变得如此简单。【QQ群：324780204、145799952】

[访问地址：](https://gitee.com/VCS/renren-security)<https://gitee.com/VCS/renren-security>



>Guns基于SpringBoot 2，致力于做更简洁的后台管理系统，完美整合springmvc + shiro + mybatis-plus + beetl!Guns项目代码简洁，注释丰富，上手容易，同时Guns包含许多基础模块(用户管理，角色管理，部门管理，字典管理等10个模块)，可以直接作为一个后台管理系统的脚手架!

[访问地址：](https://gitee.com/VCS/guns)<https://gitee.com/VCS/guns>



>wxmcs参照了很多开源的cms平台，本着快速开发，易于扩展的理念想做一款通用cms。因而最终选择基于jfinal框架使用maven多模块方式构建开发了一款内容发布系统。

[访问地址:](https://gitee.com/VCS/wxcms)<https://gitee.com/VCS/wxcms>



>central-platform简称CP，基于Spring Cloud(Finchley.RELEASE) 、Spring Boot(2.0.1)、Spring Security jwt开发 基于layui前后分离的开发平台,其中包括Gateway网关、Oauth认证服务、User用户服务、 Eureka注册中心等多个服务, 为微服务开发所需配置管理、服务发现、断路器、智能路由、 微代理等,努力为企业级打造最全面的微服务开发解决方案;

[访问地址：](https://gitee.com/VCS/central-platform)<https://gitee.com/VCS/central-platform>



>SpringBlade是一个基于Spring+SpringMVC+Beetl+Beetlsql+Shiro的开发框架。具有权限管理，多角色，父子角色，权限代理的功能，权限控制精确到按钮。

[访问地址:](https://gitee.com/VCS/SpringBlade)<https://gitee.com/VCS/SpringBlade>



> Seezoon快速开发平台基于spring mybatis shiro jquery 完全的前后端分离的后台管理系统，采用最简单技术，实现快速开发，上手门槛极低.

[访问地址：](https://gitee.com/huangdf/seezoon-framework-all)<https://gitee.com/huangdf/seezoon-framework-all>



> 实验教学平台通过校方教务处制定每个学期的教学目标，老师根据教学目标来安排教学计划和实验课程，学生根据实验课的安排进行实验课的预约，按照老师的实验安排对实验课和作业进行在线提交；老师可以在线管理学生的实验报告和实验作业并进行批改；学生可以实时查看到实验结果；教务处根据实验室及设备的占用情况和利用率进行对外开发，极大的提高是实验课的效率和利用率，同时提高了学生的动手能力。本平台适用于各类大专院校，对实验教学的管理。

[访问地址：](https://gitee.com/VCS/laboratory)<https://gitee.com/VCS/laboratory>



> dhroid 是基于android 平台, 极速开发框架，其核心设计目标是开发迅速、代码量少、学习简单、功能强大、轻量级、易扩展.使你更快,更好的开发商业级别应用

[访问地址:](https://gitee.com/VCS/dhroid)<https://gitee.com/VCS/dhroid>



>   Smart定位用当下最流行的SSM（SpringMVC + Spring + Mybatis）技术，为您构建一个易理解、高可用、高扩展性的单点登录权限管理应用基层，方便实现快速开发。权限按钮级（可控制到Controller中方法）、修改实时生效（MQ广播实现）、支持分布式（Spring + Redis提供分布式Session）。

[访问地址:](https://gitee.com/VCS/smart)<https://gitee.com/VCS/smart>



> 基于SSM+Activiti的公文管理系统的设计与实现。项目只是实现了最基本的公文流程的处理，做的不好的地方请大家谅解。项目基于eclipse开发，使用的技术：Spring、SpringMVC、Mybatis、Activiti、Maven、JackRabbit（保存上传文件）本项目以后会根据大家的意见不定期更新。

[访问地址：](https://gitee.com/VCS/article)<https://gitee.com/VCS/article>



> 宝宝喂养记是一款用Sencha Touch+PhoneGap开发的Android系统上的应用软件，可以帮助奶爸奶妈们记录初生宝宝的基本情况，如喂母乳、牛奶量和次数，大小便次数，体温，睡觉时间等信息。**可以学习其架构**

[访问地址：](https://gitee.com/VCS/WeiYang)<https://gitee.com/VCS/WeiYang>



> # [Pybbs(朋也社区)](https://tomoya92.github.io/pybbs/#/zh-cn/?id=pybbs%e6%9c%8b%e4%b9%9f%e7%a4%be%e5%8c%ba)

[访问地址：](https://github.com/PlayTaoist/pybbs)<https://github.com/PlayTaoist/pybbs>

[文档地址：](https://tomoya92.github.io/pybbs/#/zh-cn/)<https://tomoya92.github.io/pybbs/#/zh-cn/>





## 二.baomidou的Mybatis-plus

主要是用来做mybaits的数据库处理<https://mp.baomidou.com/guide/#%E7%89%B9%E6%80%A7>

有以下几个使用方便点

实现了分页功能<https://baomidou.gitee.io/mybatis-plus-doc/#/page-plugin>

文档比较齐全

支持 读写分离&&多数据源<https://baomidou.gitee.io/mybatis-plus-doc/#/multi-datasource>

自定义 SQL 方法如何使用 Wrapper<https://baomidou.gitee.io/mybatis-plus-doc/#/wrapper>





==================================================================

[博客地址](https://www.codepeople.cn)：<https://www.codepeople.cn>

==================================================================

微信公众号：

![](https://www.codepeople.cn/imges/weixin_icon/weixin.jpg)