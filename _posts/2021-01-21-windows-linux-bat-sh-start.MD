---
title:  "Windows 10和Linux脚本启动jar包服务器，并设置开机启动"
categories: Windows10 CentOS jar
tags: Windows10 CentOS jar
author: LIUREN
---

## Windows 10和Linux脚本启动jar包服务器，并设置开机启动

> 现在好多Java服务和应用都是用jar启动，jar打包成的里面会直接带有tomcat，所以可以直接启动，但是如何通过脚本启动并且设置成开机自启动
>
> 这样可以省很多事情。方便省事
>
> 协议：CC BY-SA 4.0 https://creativecommons.org/licenses/by-sa/4.0/  
>
> 版权声明：本文为原创文章，遵循 CC 4.0 BY-SA 版权协议，转载请附上原文出处链接及本声明。



##  一、Windows10脚本自启动

> 1.前提是Java环境已经配置好，就是JDK已经安装完毕

Windows脚本

1.启动jar脚本

start.bat

```shell
@echo off
java -jar E:\java\mina_server_jar\mina_server.jar
```

2.停止jar脚本(如：jar包端口9090)

stop.bat

```powershell
@echo off
set port=9090
for /f "tokens=1-5" %%i in ('netstat -ano^|findstr ":%port%"') do (
 echo kill the process %%m who use the port 
 taskkill /pid %%m -t -f
 goto q
)
:q
```

3.重启jar脚本(就是上面两个合起来)

restart.bat

```powershell
@echo off
set port=9090
for /f "tokens=1-5" %%i in ('netstat -ano^|findstr ":%port%"') do (
 echo kill the process %%m who use the port 
 taskkill /pid %%m -t -f
 goto start
)
:start
java -jar E:\java\mina_server_jar\mina_server.jar
```

如何把bat服务加入到开机启动的行列

如果是想开机就启动的话有个简单的方法

C：\ProgramData\Microsoft\Windows\Start Menu\Programs\StartUp 将bat放入这里就可以了。



只想用任务计划启动的话，继续往下看

## 二、配置系统的计划任务

打开win10左下角的菜单栏，在搜索框内输入“任务计划”

![001](http://www.codepeople.cn/imges/00x64.png)



打开任务计划后，在右边“操作”中找到“创建任务”

名称和描述随意输入，选择用户登录时运行，并且配置系统选择win10



![002](http://www.codepeople.cn/imges/00x65.png)



切换选项卡到触发器，新建触发器，选择登录时，所有用户，然后确定



![003](http://www.codepeople.cn/imges/00x66.png)



切换选项卡到“操作”，新建，操作选择启动程序，程序或者脚本填我们前面写好的.bat文件路径，然后确定



![003](http://www.codepeople.cn/imges/00x67.png)



后面两个选项卡“条件”和“设置”是更加细粒度的条件设置，其中，在设置中，“允许按需允许任务”要打勾，方便测试，

其他的可以根据需求去设置，到这里已经满足我们的需求了，就不往下做了

到主面板就可以看到我们新增的计划了



![003](http://www.codepeople.cn/imges/00x68.png)



右击任务，选择启动，弹出启动界面

注意：360或者安全管家记得放行，因为这些安全程序都提示拦截，记得放行，不要禁用

##  三、CentOS7.4中使用脚本启动jar文件

启动脚本

start.sh

```
nohup java -Xmx2g -Xms2g -Xss256k -XX:+DisableExplicitGC  -XX:+UseConcMarkSweepGC -XX:+CMSParallelRemarkEnabled -XX:LargePageSizeInBytes=128m -XX:+UseFastAccessorMethods -XX:+UseCMSInitiatingOccupancyOnly -XX:CMSInitiatingOccupancyFraction=70 -Duser.timezone=GMT+8 -jar app-sdt.jar --spring.profiles.active=prod >/dev/null 2>&1 &

```

或者使用全量脚本

app-service.sh

```shell
#!/bin/bash
#source /etc/profile
# Auth：Liucx
# Please change these parameters according to your real env.
# 请把JAVA_HOME改成你自己的环境路径.
# 如果需要修改启动所需的配置文件修改 -Dspring.profiles.active=dev 指定环境.
# set Java Home: Remember that dolphin only supports JDK8!
JAVA_HOME=/usr/java/jdk1.8.0_211
#JAVA_HOME=/usr/local/java/jdk1.8.0_11
# application directory
cd `dirname $0`
APP_HOME=`pwd`
APP_NAME="`cd ${APP_HOME} && find -name '*.jar' `"
JAR_HOME_TOW=${APP_NAME:2}
JAR_HOME=${JAR_HOME_TOW%%.*}

echo "=============================================="
echo "========   欢迎使用科澜脚本启动jar    ========"
echo "========   生成的日志在本目录下       ========"
echo "========   日志是$JAR_HOME.log         ========"
echo "=============================================="

# JAR_HOME=`pwd |awk -F"/" '{print $NF}'`
# Java JVM lunch parameters
if [ x"$JAVA_MEM_OPTS" == x ];then
  JAVA_MEM_OPTS="-server -Xms512m -Xmx512m -Xmn128m -Xss256k -XX:+DisableExplicitGC -XX:+UseConcMarkSweepGC -XX:+CMSParallelRemarkEnabled -XX:LargePageSizeInBytes=128m -XX:+UseFastAccessorMethods -XX:+UseCMSInitiatingOccupancyOnly -XX:CMSInitiatingOccupancyFraction=70 "
fi
# waiting timeout for starting, in seconds
START_WAIT_TIMEOUT=30
psid=0
#/usr/java/jdk1.8.0_211/bin/jps -l | grep test
checkpid() {
  javaps=`$JAVA_HOME/bin/jps -l | grep "$JAR_HOME"`
  if [ -n "$javaps" ]; then
   psid=`echo $javaps | awk '{print $1}'`
  else
   psid=0
  fi
}
start() {
  checkpid
  if [ $psid -ne 0 ]; then
   echo "================================"
   echo "warn: $APP_NAME already started! (pid=$psid)"
   echo "================================"
  else
   echo -n "Starting $APP_NAME ..."
   #-Dspring.profiles.active=dev 指定环境
   nohup $JAVA_HOME/bin/java -jar -Dspring.profiles.active=prod -DCONFIG_ENABLE=true $JAVA_MEM_OPTS ${APP_NAME} >${JAR_HOME}.log 2>&1 &  
   sleep 5
   checkpid
   echo ">>>>>>>>>(pid=$psid) [OK]"
   if [ $psid -ne 0 ]; then
     echo "(pid=$psid) [OK]"
   else
     echo "[Failed]"
   fi
  fi
}
stop() {
  sleep 2
  checkpid
  if [ $psid -ne 0 ]; then
   echo -n "Stopping $APP_NAME ...(pid=$psid) "
   #kill -9 $psid
   kill -9 $psid
   if [ $? -eq 0 ]; then
     echo "[OK]"
   else
     echo "[Failed]"
   fi
#   checkpid
#   if [ $psid -ne 0 ]; then
#     stop
#   fi
  else
   echo "================================"
   echo "warn: $APP_NAME is not running"
   echo "================================"
  fi
}
status() {
  checkpid
  if [ $psid -ne 0 ]; then
   echo "$APP_NAME is running! (pid=$psid)"
  else
   echo "$APP_NAME is not running"
  fi
}
info() {
  echo "System Information:"
  echo "****************************"
  echo `head -n 1 /etc/issue`
  echo `uname -a`
  echo
  echo "JAVA_HOME=$JAVA_HOME"
  echo `$JAVA_HOME/bin/java -version`
  echo
  echo "APP_HOME=$APP_HOME"
  echo "APP_NAME=$APP_NAME"
  echo "****************************"
}
case "$1" in
  'start')
   start
   ;;
  'stop')
   stop
   ;;
  'restart')
   stop
   echo "You restart the application ..."
   start #
   ;;
  'status')
   status
   ;;
  'info')
   info
   ;;
 *)
   echo "启动方式: $0 {start|stop|restart|status|info}"
   exit 1
esac
```

使用方法：

直接和需要启动jar项目放到同一个目录下。然后给该脚本权限

```shell
chmod 755 app-service.sh
```

启动，停止，重启，服务信息，服务状态使用方法如下：

```powershell
./app-service.sh start|stop|restart|info|stauts
```



linux设置开机启动：

```
mv start.sh  /etc/init.d
chmod +x start.sh
chkconfig --add servicename
```





博客地址：<https://www.codepeople.cn>

=====================================================================

微信公众号：

![](https://www.codepeople.cn/imges/weixin_icon/weixin.jpg)