---
title:  "windows_查看端口被占用情况、结束进程"
categories: windows port
tags: windows port
author: LIUREN
---

# windows_查看端口被占用情况、结束进程

> 
>



## 第一步　查看指定端口占用情况（端口引号有没有都可以，此处以5040为例）

```
netstat -aon|findstr 5040
```

![](https://www.codepeople.cn/imges/00x151.png)

记住最后一列数字PID，此处为25428。

## 第二步  根据PID查看相应的进程或程序

```shell
tasklist|findstr 25428
```



## 第三步  任务管理器查看

快捷键Ctrl+Alt+Delete，打开任务管理器，可查看PID对应的进程

![](https://www.codepeople.cn/imges/00x152.png)

## 第四步  手动结束该进程



> 参考文章：<https://zhuanlan.zhihu.com/p/141349611>
>

