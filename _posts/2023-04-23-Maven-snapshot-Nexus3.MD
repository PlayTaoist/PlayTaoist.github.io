---
title:  "Maven手动上传snapshot版本的jar包到maven私服Nexus3"
categories: Maven snapshot Nexus3
tags: Maven snapshot Nexus3
author: LIUREN
---

# Maven手动上传snapshot版本的jar包到maven私服Nexus3

> 启动服务发现少包，然后百度一下 手动命令行上传 snapshot的jar包。
>



## 第一步　问为啥手动。因为页面上不允许上传snapshot的包。会报如下错误

```
Version policy mismatch, cannot upload SNAPSHOT content to RELEASE repositories for file '0'
```



## 第二步  	Maven环境准备

```shell
下载maven安装包，解压后配置好path环境变量。具体配置请参考百度
```



## 第三步  	配置Maven的settings.xml

```shell
 <!-- 在settings标签下配置以下内容 -->
  <servers>
    <server>
      <!--repositoryId，上传SNAPSHOT jar时需要指定此id -->
      <id>nexus-snapshot</id>
      <!--nexus 账号密码  -->
      <username>admin</username>
      <password>admin123</password>
    </server>
  </servers>
```



## 第四步  	上传Nexus3 本地jar和pom.xml

##### 4.1 用管理员权限打开cmd命令窗口，cd到在第三方jar包所在目录下，输入以下命令（如果打开的时powershell ，键入cmd字符回车切换到cmd命令下执行脚本）

##### 4.2 上传jar包和pom文件命令

```shell
mvn deploy:deploy-file^
 -DgroupId=com.cloud.infinite^
 -DartifactId=base-exp^
 -Dversion=1.3.2-SNAPSHOT^
 -Dpackaging=jar^
 -Dfile=base-exp-1.3.2-SNAPSHOT.jar^
 -DpomFile=base-exp-1.3.2-SNAPSHOT.pom^
 -Durl=http://192.168.1.189:8081/repository/maven-snapshots/^
 -DrepositoryId=nexus-snapshot -X
```

```shell
有时候只需要上传pom文件(如maven父工程)，使用以下命令即可
mvn deploy:deploy-file^
-DgroupId=com.cloud.infinite^
-DartifactId=base-plugin^
-Dversion=1.3.2-SNAPSHOT^
-Dpackaging=pom^
-Dfile=base-plugin-1.3.2-SNAPSHOT.pom^
-Durl=http://192.168.1.189:8081/repository/maven-snapshots/^
-DrepositoryId=nexus-snapshot -X
```

##### 4.3 命令参数简介

```shell

-DgroupId=com.aaa //com.aaa为要上传的jar包groupId
-Dversion=0.0.1-SNAPSHOT //0.0.1-SNAPSHOT为要上传的jar包版本号
-Dpackaging=jar //上传文件类型，pom或者jar
-Dfile=bbb-0.0.1-SNAPSHOT.jar //bbb-0.0.1-SNAPSHOT.jar为要上传的jar包/pom名称
-DpomFile=bbb-0.0.1-SNAPSHOT.pom  //要上传的pom文件
-Durl=http://192.168.1.189:8081/repository/maven-snapshots/ //远程Nexus私服仓库地址，可在nexus web浏览仓库页面，点击copy URL的按钮获取到
-DrepositoryId=nexus-snapshot //nexus-snapshot  为我们第2步配置的本地maven conf/settings.xml文件中其中一个server的id，nexus上传jar需要用这种方式验证账号密码
-X 				// debug模式执行mvn命令，方便查看异常日志
```

##### 4.4 常见错误：

```shell
[ERROR] The goal you specified requires a project to execute but there is no POM in this directory...
```

解决方案：cmd命令必须用管理员权限打开，如果打开的是powershell，请输入`cmd`命令切换到cmd下执行

```shell
[ERROR] Failed to execute goal org.apache.maven.plugins:maven-deploy-plugin:2.7:deploy-file (default-cli) on project standalone-pom: Cannot deploy artifact from the local repository
```

解决方案：不能在本地maven仓库中直接运行上传命令，jar包需要挪动.m2/repository目录外上传，或者重命名目录repository

##### 4.5 注意事项

```shell
上传本地jar包时最好同时上传对应的pom文件，否则jar包依赖不会传递
```



## 第五步   最终命令行

```shell
mvn deploy:deploy-file^ -DgroupId=com.jdcloud.apigateway^ -DartifactId=sdk^ -Dversion=0.1.0-SNAPSHOT^ -Dpackaging=jar^ -Dfile=sdk-0.1.0-SNAPSHOT.jar^ -DpomFile=sdk-0.1.0-SNAPSHOT.pom^ -Durl=http://248.248.248.248:8081/repository/maven-snapshots/^ -DrepositoryId=releases -X
```



> 参考文章：<https://blog.csdn.net/haohaifeng002/article/details/112471759>
>

