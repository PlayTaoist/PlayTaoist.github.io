---
title:  "从零开始学springboot-数据库版本管理Flyway"
categories: rtsp
tags: rtsp
author: LIUREN
---

# 从零开始学springboot数据库版本管理Flyway

> 一个项目上线后,在不断的迭代过程中,总会出于这样那样的原因需要对数据库表做“增删改”这些“硬”操作.
>
> 事实上,在项目真正稳定运行前,对数据库的调整根本不可能停止.
>
> 那么,每次发版前,收集每个开发人员调整的SQL文件就慢慢的变成一件麻烦的事,大概会有如下痛点:
>
> - 开发人员往往会遗忘SQL更改
> - 每个人的SQL会有“先后”执行的顺序问题
> - 需要手动去数据库执行
>
> 那么,有没有这么一套“工具”可以让执行SQL变更脚本变得更加自动,智能呢?
>
> 本章,我们介绍数据库版本管理工具Flyway工具
>
> 作者：MrCoderStack
>
> 链接：https://www.jianshu.com/p/54de82c20151
>
> 来源：简书
>
> 著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。



## Flyway简介

```undefined
Flyway是独立于数据库的应用、管理并跟踪数据库变更的数
据库版本管理工具。用通俗的话讲，Flyway可以像Git管理不
同人的代码那样，管理不同人的sql脚本，从而做到数据库同
步
```

**Flyway流程**

- 首先配置好flyway的基本信息后，运行项目，会在数据库表中默认新建一个数据表用于存储flyway的运行信息，默认的数据库名：flyway_schema_history
- 紧接着Flyway将开始扫描文件系统或应用程序的类路径进行迁移。然后，Flyway的数据迁移将基于对用sql脚本的版本号进行排序，并按顺序应用：

可以看到执行数据库表后在checksum中储存一个数值，用于在之后运行过程中对比sql文件执行是否有变化。

**注意：**
 flyway在执行脚本时，会在源数据表中检查checksum值，并确定上次运行到哪一个脚本文件，本次执行时从下一条脚本文件开始执行。所以编写脚本的时候不要去修改原有的脚本内容，并且新的脚本版本号要连续

接下来,我们使用Springboot集成Flyway

## Springboot 集成Flyway

- 老规矩,我们先创建一个Springboot项目(自行集成mybatis)
   然后在pom.xml加入如下依赖集成Flyway

```xml
        <dependency>
            <groupId>org.flywaydb</groupId>
            <artifactId>flyway-core</artifactId>
        </dependency>
```

- 然后我们在application.yml中写入mysql的配置及Flyway的配置(Flyway配置可不写,不写配置的话默认读取当前项目下的 “resources/db/migration”目录)
```yaml
spring:
  datasource:
    url: jdbc:mysql://127.0.0.1:3306/test?useUnicode=true&characterEncoding=UTF-8&useSSL=false&autoReconnect=true&failOverReadOnly=false&serverTimezone=GMT%2B8
    username: root
    password: 123456
    driver-class-name: com.mysql.cj.jdbc.Driver
  flyway:
    locations: classpath:db/migration/
```

- 接下来,我们在resources/db/migration这个目录下创建我们需要执行的SQL脚本即可
   此处,注意,SQL脚本命名要按照规则:



  ![img](https:////upload-images.jianshu.io/upload_images/16747124-b22a65a3ad3c5e94.png?imageMogr2/auto-orient/strip|imageView2/2/w/1067/format/webp)

  flyway.png

```css
sql脚本的格式：V_版本号__版本描述.sql  (中间是两个下划线)
形如:
V1_1__init.sql        对应的版本号就是1.1
V1_2__update       对应的版本号就是1.2
```



![img](https:////upload-images.jianshu.io/upload_images/16747124-83ecbdaeb6a22506.png?imageMogr2/auto-orient/strip|imageView2/2/w/630/format/webp)



## 运行

运行,我们就会发现数据库会多出三张表



![img](https:////upload-images.jianshu.io/upload_images/16747124-e10e76ba1973a514.png?imageMogr2/auto-orient/strip|imageView2/2/w/550/format/webp)

image.png

book和books是我们写的脚本创建的两张表,而flyway_schema_history此表为Flyway用来记录版本信息的默认表



![img](https:////upload-images.jianshu.io/upload_images/16747124-9e66e4ff4029a30e.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

image.png

## 附录-Flyway其它配置

```csharp
flyway.baseline-description对执行迁移时基准版本的描述.
flyway.baseline-on-migrate当迁移时发现目标schema非空，而且带有没有元数据的表时，是否自动执行基准迁移，默认false.
flyway.baseline-version开始执行基准迁移时对现有的schema的版本打标签，默认值为1.
flyway.check-location检查迁移脚本的位置是否存在，默认false.
flyway.clean-on-validation-error当发现校验错误时是否自动调用clean，默认false.
flyway.enabled是否开启flywary，默认true.
flyway.encoding设置迁移时的编码，默认UTF-8.
flyway.ignore-failed-future-migration当读取元数据表时是否忽略错误的迁移，默认false.
flyway.init-sqls当初始化好连接时要执行的SQL.
flyway.locations迁移脚本的位置，默认db/migration.
flyway.out-of-order是否允许无序的迁移，默认false.
flyway.password目标数据库的密码.
flyway.placeholder-prefix设置每个placeholder的前缀，默认${.
flyway.placeholder-replacementplaceholders是否要被替换，默认true.
flyway.placeholder-suffix设置每个placeholder的后缀，默认}.
flyway.placeholders.[placeholder name]设置placeholder的value
flyway.schemas设定需要flywary迁移的schema，大小写敏感，默认为连接默认的schema.
flyway.sql-migration-prefix迁移文件的前缀，默认为V.
flyway.sql-migration-separator迁移脚本的文件名分隔符，默认__
flyway.sql-migration-suffix迁移脚本的后缀，默认为.sql
flyway.tableflyway使用的元数据表名，默认为schema_version
flyway.target迁移时使用的目标版本，默认为latest version
flyway.url迁移时使用的JDBC URL，如果没有指定的话，将使用配置的主数据源
flyway.user迁移数据库的用户名
flyway.validate-on-migrate迁移时是否校验，默认为true.
```





博客地址：<https://www.codepeople.cn>

=====================================================================

微信公众号：

![](https://www.codepeople.cn/imges/weixin_icon/weixin.jpg)