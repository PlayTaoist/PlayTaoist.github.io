---
title:  "SpringBoot2.0 打包和部署方法"
categories: SpringBoot2.0
tags: SpringBoot2.0
author: LIUREN
---

# SpringBoot2.0 打包和部署方法

>SpringBoot2.0 打包和部署方法,SpringBoot可以打包成war包也可以直接打包成jar包进行运行



## 投产上线

其实我觉得这个阶段，应该还是比较简单一般分为两种；一种是打包成jar包直接执行，另一种是打包成war包放到tomcat服务器下。

### 打成jar包

如果你使用的是maven来管理项目，执行以下命令既可以

```shell
cd 项目跟目录（和pom.xml同级）
mvn clean package
## 或者执行下面的命令
## 排除测试代码后进行打包
mvn clean package  -Dmaven.test.skip=true
```

打包完成后jar包会生成到target目录下，命名一般是 项目名+版本号.jar

启动jar包命令

```shell
java -jar  target/spring-boot-scheduler-1.0.0.jar
```

这种方式，只要控制台关闭，服务就不能访问了。下面我们使用在后台运行的方式来启动:

```shell
nohup java -jar target/spring-boot-scheduler-1.0.0.jar &
```

也可以在启动的时候选择读取不同的配置文件

```shell
java -jar app.jar --spring.profiles.active=dev
```

也可以在启动的时候设置jvm参数

```shell
java -Xms10m -Xmx80m -jar app.jar &
```

**gradle**
如果使用的是gradle,使用下面命令打包

```shell
gradle build
java -jar build/libs/mymodule-0.0.1-SNAPSHOT.jar
```

### 打成war包

打成war包一般可以分两种方式来实现，第一种可以通过eclipse这种开发工具来导出war包，另外一种是使用命令来完成，这里主要介绍后一种

1、maven项目，修改pom包

将

```shell
<packaging>jar</packaging>
```

改为

```shell
<packaging>war</packaging>
```

2、打包时排除tomcat.

```shell
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-tomcat</artifactId>
	<scope>provided</scope>
</dependency>
```

在这里将scope属性设置为provided，这样在最终形成的WAR中不会包含这个JAR包，因为Tomcat或Jetty等服务器在运行时将会提供相关的API类。

3、注册启动类

创建ServletInitializer.java，继承SpringBootServletInitializer ，覆盖configure()，把启动类Application注册进去。外部web应用服务器构建Web Application Context的时候，会把启动类添加进去。

```java
public class ServletInitializer extends SpringBootServletInitializer {
    @Override
    protected SpringApplicationBuilder configure(SpringApplicationBuilder application) {
        return application.sources(Application.class);
    }
}
```

最后执行

```shell
mvn clean package  -Dmaven.test.skip=true
```

会在target目录下生成：项目名+版本号.war文件，拷贝到tomcat服务器中启动即可。

**gradle**

如果使用的是gradle,基本步奏一样，build.gradle中添加war的支持，排除spring-boot-starter-tomcat：

```groovy
...

apply plugin: 'war'

...

dependencies {
    compile("org.springframework.boot:spring-boot-starter-web:1.4.2.RELEASE"){
    	exclude mymodule:"spring-boot-starter-tomcat"
    }
}
...
```

再使用构建命令

```groovy
gradle build
```

war会生成在build\libs 目录下。

## 生产运维

### 查看JVM参数的值

可以根据java自带的jinfo命令：

```
jinfo -flags pid
```

来查看jar 启动后使用的是什么gc、新生代、老年代分批的内存都是多少，示例如下：

```
-XX:CICompilerCount=3 -XX:InitialHeapSize=234881024 -XX:MaxHeapSize=3743416320 -XX:MaxNewSize=1247805440 -XX:MinHeapDeltaBytes=524288 -XX:NewSize=78118912 -XX:OldSize=156762112 -XX:+UseCompressedClassPointers -XX:+UseCompressedOops -XX:+UseFastUnorderedTimeStamps -XX:+UseParallelGC
```

- `-XX:CICompilerCount` ：最大的并行编译数
- `-XX:InitialHeapSize` 和 `-XX:MaxHeapSize` ：指定JVM的初始和最大堆内存大小
- `-XX:MaxNewSize` ： JVM堆区域新生代内存的最大可分配大小
- …
- `-XX:+UseParallelGC` ：垃圾回收使用Parallel收集器

### 如何重启

**简单粗暴**

直接kill掉进程再次启动jar包

```
ps -ef|grep java 
##拿到对于Java程序的pid
kill -9 pid
## 再次重启
Java -jar  xxxx.jar
```

当然这种方式比较传统和暴力，所以建议大家使用下面的方式来管理

**脚本执行**

如果使用的是maven,需要包含以下的配置

```xml
<plugin>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-maven-plugin</artifactId>
    <configuration>
        <executable>true</executable>
    </configuration>
</plugin>
```

如果使用是gradle，需要包含下面配置

```groovy
springBoot {
    executable = true
}
```

启动方式：

1、 可以直接`./yourapp.jar` 来启动

2、注册为服务

也可以做一个软链接指向你的jar包并加入到`init.d`中，然后用命令来启动。

init.d 例子:

```shell
ln -s /var/yourapp/yourapp.jar /etc/init.d/yourapp
chmod +x /etc/init.d/yourapp
```

这样就可以使用`stop`或者是`restart`命令去管理你的应用。

```shell
/etc/init.d/yourapp start|stop|restart
```

或者

```shell
service yourapp start|stop|restart
```

到此 springboot项目如何测试、联调和打包投产均已经介绍完，以后可以找时间研究一下springboot的自动化运维，以及spring boot 和docker相结合的使用。