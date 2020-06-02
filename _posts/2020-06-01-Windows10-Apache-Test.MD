---
title:  "Windows10中Apache ab并发负载压力测试"
categories: Windows10 Apache
tags: Windows10 Apache
author: LIUREN
---

# Windows10中Apache ab并发负载压力测试

> Apache的ab命令模拟多线程并发请求，测试服务器负载压力，也可以测试nginx、lighthttp、IIS等其它Web服务器的压力。 
> ab命令对发出负载的计算机要求很低，既不会占用很多CPU，也不会占用太多的内存，但却会给目标服务器造成巨大的负载，因此是某些DDOS攻击之必备良药，老少皆宜。自己使用也须谨慎。否则一次上太多的负载，造成目标服务器直接因内存耗光死机，而不得不硬重启，得不偿失。
>
> 在带宽不足的情况下，最好是本机进行测试，建议使用内网的另一台或者多台服务器通过内网进行测试，这样得出的数据，准确度会高很多。远程对web服务器进行压力测试，往往效果不理想（因为网络延时过大或带宽不足）
>
> 协议：CC BY-SA 4.0 https://creativecommons.org/licenses/by-sa/4.0/  
>
> 版权声明：本文为原创文章，遵循 CC 4.0 BY-SA 版权协议，转载请附上原文出处链接及本声明。



## 第一步：下载

下载地址如下：<https://www.apachehaus.com/cgi-bin/download.plx>

![](https://www.codepeople.cn/imges/00x34.png)

解压 apache2.4目录下

![](https://www.codepeople.cn/imges/00x35.png)

修改配置文件httpd.conf

![](https://www.codepeople.cn/imges/00x36.png)

**参数文档：** 
<http://httpd.apache.org/docs/2.2/programs/ab.html>



**运行：** 在Windows系统下，打开cmd命令行窗口，定位到apache安装目录的bin目录下 
cd C:\Program Files (x86)\Apache Software Foundation\Apache2.2\bin

键入命令： 
ab -n 800 -c 800  <http://192.168.0.10/> 
（-n发出800个请求，-c模拟800并发，相当800人同时访问，后面是测试url）

ab -t 60 -c 100 <http://192.168.0.10/> 
在60秒内发请求，一次100个请求。 

//如果需要在url中带参数，这样做 
ab -t 60 -c 100 -T "text/plain" -p p.txt <http://192.168.0.10/hello.html> 
p.txt 是和ab.exe在一个目录 
p.txt 中可以写参数，如  p=wdp&fq=78 





**结果参数解释：** 
This is ApacheBench, Version 2.3 <$Revision: 655654 $> 
Copyright 1996 Adam Twiss, Zeus Technology Ltd, <http://www.zeustech.net/> 
Licensed to The Apache Software Foundation, <http://www.apache.org/>

Benchmarking 192.168.0.10 (be patient) 
Completed 100 requests 
Completed 200 requests 
Completed 300 requests 
Completed 400 requests 
Completed 500 requests 
Completed 600 requests 
Completed 700 requests 
Completed 800 requests 
Finished 800 requests


Server Software:        Microsoft-HTTPAPI/2.0 
Server Hostname:        192.168.0.10 
Server Port:            80

Document Path:          / 
Document Length:        315 bytes       HTTP响应数据的正文长度

Concurrency Level:      800 
Time taken for tests:   0.914 seconds    所有这些请求处理完成所花费的时间 
Complete requests:      800             完成请求数 
Failed requests:        0                失败请求数 
Write errors:           0                
Non-2xx responses:      800 
Total transferred:      393600 bytes     网络总传输量 
HTML transferred:       252000 bytes     HTML内容传输量 
Requests per second:    875.22 [#/sec] (mean) 吞吐量-每秒请求数 
Time per request:       914.052 [ms] (mean)  服务器收到请求，响应页面要花费的时间 
Time per request:       1.143 [ms] (mean, across all concurrent requests) 并发的每个请求平均消耗时间 
Transfer rate:          420.52 [Kbytes/sec] received 平均每秒网络上的流量，可以帮助排除是否存在网络流量过大导致响应时间延长的问题


网络上消耗的时间的分解： 
Connection Times (ms) 
​              min  mean[+/-sd] median   max 
Connect:        0    1   0.5      1       3 
Processing:   245  534 125.2    570     682 
Waiting:       11  386 189.1    409     669 
Total:        246  535 125.0    571     684

整个场景中所有请求的响应情况。在场景中每个请求都有一个响应时间 
其中 50％ 的用户响应时间小于 571 毫秒 
80 ％ 的用户响应时间小于 652 毫秒 
最大的响应时间小于 684 毫秒 
Percentage of the requests served within a certain time (ms) 
  50%    571 
  66%    627 
  75%    646 
  80%    652 
  90%    666 
  95%    677 
  98%    681 
  99%    682 
100%    684 (longest request)

参考地址：<https://www.cnblogs.com/lishuyi/p/5808661.html>



博客地址：<https://www.codepeople.cn>

=====================================================================

微信公众号：

![](https://www.codepeople.cn/imges/weixin_icon/weixin.jpg)