---
title:  "GoLand永久激活"
categories: go
tags: go
author: LIUREN
---

# GoLand永久激活

> Go 是一个开源的编程语言，它能让构造简单、可靠且高效的软件变得容易。Go 语言被设计成一门应用于搭载 Web 服务器，存储集群或类似用途的巨型中央服务器的系统编程语言。对于高性能分布式系统领域而言，Go 语言无疑比大多数其它语言有着更高的开发效率。它提供了海量并行的支持，这对于游戏服务端的开发而言是再好不过了。



###  一、版本支持

```
此教程已支持最新2019.2版本
```

### 二、下载新版破解补丁

点击链接: https://pan.baidu.com/s/10QUhlknMFsUoQBKOhbGD0g 提取码: tq6p 下载补丁文件 `jetbrains-agent.jar` 并将它放置到 Goland安装目录的\bin目录下（位置可随意，放这里是怕误操作删除了破解文件）。


![](https://www.codepeople.cn/imges/go/014.png)

### 三、进入项目界面

如果你之前已经使用有效期激活过可跳过此步骤，如果你是刚下载的GoLand，则需要点击激活窗口的“Evaluate for free”免费试用，然后再创建一个空项目，这样就可以进入到GoLand的工作页面

![](https://www.codepeople.cn/imges/go/015.jpg)

![](https://www.codepeople.cn/imges/go/016.jpg)

![](https://www.codepeople.cn/imges/go/017.jpg)

### 四、修改配置文件

进入到项目界面后，点击GoLand最上面的菜单栏中的 “Help” -> “Edit Custom VM Options …”，如果提示是否要创建文件，请点”Yes”。

![](https://www.codepeople.cn/imges/go/018.png)

在打开的vmoptions编辑窗口末行添加：`-javaagent:你goland的安装目录\jetbrains-agent.jar`
![](https://www.codepeople.cn/imges/go/019.png)

我自己的路径是`-javaagent:D:\GoLand\go_idea\GoLand 2019.2.1\bin\jetbrains-agent.jar`

请仔细检查补丁路径是否正确，如果错误则会出现GoLand打不开的情况，这时候可以删除用户配置目录下的goland文件夹:

windwos：C:\Users\用户名\
macos：~/Library/Preferences/
ubuntu：~/.

![](https://www.codepeople.cn/imges/go/020.png)

**修改完配置文件之后重启GoLand**

**修改完配置文件之后重启GoLand**

**修改完配置文件之后重启GoLand**

### 五、输入激活码

重启GoLand之后，点击菜单栏中的 “Help” -> “Register …”，这里有两种激活方式：

一、选择最后一种**License server**激活方式，地址填入：`http://jetbrains-license-server` （应该会自动填上），或者点击按钮：”Discover Server”来自动填充地址，完成激活

![](https://www.codepeople.cn/imges/go/021.png)

二、如果服务器激活方式无法激活，还可以选择**Activation code**方式激活，复制下面激活码即可
![](https://www.codepeople.cn/imges/go/022.png)

激活码：

```shell
JQE11SV0BR-eyJsaWNlbnNlSWQiOiJKUUUxMVNWMEJSIiwibGljZW5zZWVOYW1lIjoicGlnNiIsImFzc2lnbmVlTmFtZSI6IiIsImFzc2lnbmVlRW1haWwiOiIiLCJsaWNlbnNlUmVzdHJpY3Rpb24iOiIiLCJjaGVja0NvbmN1cnJlbnRVc2UiOmZhbHNlLCJwcm9kdWN0cyI6W3siY29kZSI6IklJIiwiZmFsbGJhY2tEYXRlIjoiMjA4OS0wNy0wNyIsInBhaWRVcFRvIjoiMjA4OS0wNy0wNyJ9LHsiY29kZSI6IkFDIiwiZmFsbGJhY2tEYXRlIjoiMjA4OS0wNy0wNyIsInBhaWRVcFRvIjoiMjA4OS0wNy0wNyJ9LHsiY29kZSI6IkRQTiIsImZhbGxiYWNrRGF0ZSI6IjIwODktMDctMDciLCJwYWlkVXBUbyI6IjIwODktMDctMDcifSx7ImNvZGUiOiJQUyIsImZhbGxiYWNrRGF0ZSI6IjIwODktMDctMDciLCJwYWlkVXBUbyI6IjIwODktMDctMDcifSx7ImNvZGUiOiJHTyIsImZhbGxiYWNrRGF0ZSI6IjIwODktMDctMDciLCJwYWlkVXBUbyI6IjIwODktMDctMDcifSx7ImNvZGUiOiJETSIsImZhbGxiYWNrRGF0ZSI6IjIwODktMDctMDciLCJwYWlkVXBUbyI6IjIwODktMDctMDcifSx7ImNvZGUiOiJDTCIsImZhbGxiYWNrRGF0ZSI6IjIwODktMDctMDciLCJwYWlkVXBUbyI6IjIwODktMDctMDcifSx7ImNvZGUiOiJSUzAiLCJmYWxsYmFja0RhdGUiOiIyMDg5LTA3LTA3IiwicGFpZFVwVG8iOiIyMDg5LTA3LTA3In0seyJjb2RlIjoiUkMiLCJmYWxsYmFja0RhdGUiOiIyMDg5LTA3LTA3IiwicGFpZFVwVG8iOiIyMDg5LTA3LTA3In0seyJjb2RlIjoiUkQiLCJmYWxsYmFja0RhdGUiOiIyMDg5LTA3LTA3IiwicGFpZFVwVG8iOiIyMDg5LTA3LTA3In0seyJjb2RlIjoiUEMiLCJmYWxsYmFja0RhdGUiOiIyMDg5LTA3LTA3IiwicGFpZFVwVG8iOiIyMDg5LTA3LTA3In0seyJjb2RlIjoiUk0iLCJmYWxsYmFja0RhdGUiOiIyMDg5LTA3LTA3IiwicGFpZFVwVG8iOiIyMDg5LTA3LTA3In0seyJjb2RlIjoiV1MiLCJmYWxsYmFja0RhdGUiOiIyMDg5LTA3LTA3IiwicGFpZFVwVG8iOiIyMDg5LTA3LTA3In0seyJjb2RlIjoiREIiLCJmYWxsYmFja0RhdGUiOiIyMDg5LTA3LTA3IiwicGFpZFVwVG8iOiIyMDg5LTA3LTA3In0seyJjb2RlIjoiREMiLCJmYWxsYmFja0RhdGUiOiIyMDg5LTA3LTA3IiwicGFpZFVwVG8iOiIyMDg5LTA3LTA3In0seyJjb2RlIjoiUlNVIiwiZmFsbGJhY2tEYXRlIjoiMjA4OS0wNy0wNyIsInBhaWRVcFRvIjoiMjA4OS0wNy0wNyJ9XSwiaGFzaCI6IjEyNzk2ODc3LzAiLCJncmFjZVBlcmlvZERheXMiOjcsImF1dG9Qcm9sb25nYXRlZCI6ZmFsc2UsImlzQXV0b1Byb2xvbmdhdGVkIjpmYWxzZX0=-khgsQrnDiglknF0m+yyoYGJXX4vFE3IIVaoMd0bkpfAlMiYM4FUK1JM7uMnVSN0NBC7qtZjYlNzPscEyKE8634uGuY/uToFQnIOCtyUfBxB6j0wF/DcCjhKMNDbnJ1RKZ2VaALuC9B6d6lhtEKm9+urXWTBq7h2VfIBv5wk1Ul9T/m9Dwkz/LccTqnxO0PP288fF13ZbmcLI1/D0dqp/QxYshW6CLR+2Tvk6QCPoaOTKDU/eL1AssD7/mO1g2ZJA+k//8qfRMLgdLmLrMdyiaIhrsM/jJk2qDfTaMcCNylkWXLgKwSvEQG95IhitLN9+GQ4pBW3gOTNl82Gem7jEkA==-MIIElTCCAn2gAwIBAgIBCTANBgkqhkiG9w0BAQsFADAYMRYwFAYDVQQDDA1KZXRQcm9maWxlIENBMB4XDTE4MTEwMTEyMjk0NloXDTIwMTEwMjEyMjk0NlowaDELMAkGA1UEBhMCQ1oxDjAMBgNVBAgMBU51c2xlMQ8wDQYDVQQHDAZQcmFndWUxGTAXBgNVBAoMEEpldEJyYWlucyBzLnIuby4xHTAbBgNVBAMMFHByb2QzeS1mcm9tLTIwMTgxMTAxMIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEA5ndaik1GD0nyTdqkZgURQZGW+RGxCdBITPXIwpjhhaD0SXGa4XSZBEBoiPdY6XV6pOfUJeyfi9dXsY4MmT0D+sKoST3rSw96xaf9FXPvOjn4prMTdj3Ji3CyQrGWeQU2nzYqFrp1QYNLAbaViHRKuJrYHI6GCvqCbJe0LQ8qqUiVMA9wG/PQwScpNmTF9Kp2Iej+Z5OUxF33zzm+vg/nYV31HLF7fJUAplI/1nM+ZG8K+AXWgYKChtknl3sW9PCQa3a3imPL9GVToUNxc0wcuTil8mqveWcSQCHYxsIaUajWLpFzoO2AhK4mfYBSStAqEjoXRTuj17mo8Q6M2SHOcwIDAQABo4GZMIGWMAkGA1UdEwQCMAAwHQYDVR0OBBYEFGEpG9oZGcfLMGNBkY7SgHiMGgTcMEgGA1UdIwRBMD+AFKOetkhnQhI2Qb1t4Lm0oFKLl/GzoRykGjAYMRYwFAYDVQQDDA1KZXRQcm9maWxlIENBggkA0myxg7KDeeEwEwYDVR0lBAwwCgYIKwYBBQUHAwEwCwYDVR0PBAQDAgWgMA0GCSqGSIb3DQEBCwUAA4ICAQBonMu8oa3vmNAa4RQP8gPGlX3SQaA3WCRUAj6Zrlk8AesKV1YSkh5D2l+yUk6njysgzfr1bIR5xF8eup5xXc4/G7NtVYRSMvrd6rfQcHOyK5UFJLm+8utmyMIDrZOzLQuTsT8NxFpbCVCfV5wNRu4rChrCuArYVGaKbmp9ymkw1PU6+HoO5i2wU3ikTmRv8IRjrlSStyNzXpnPTwt7bja19ousk56r40SmlmC04GdDHErr0ei2UbjUua5kw71Qn9g02tL9fERI2sSRjQrvPbn9INwRWl5+k05mlKekbtbu2ev2woJFZK4WEXAd/GaAdeZZdumv8T2idDFL7cAirJwcrbfpawPeXr52oKTPnXfi0l5+g9Gnt/wfiXCrPElX6ycTR6iL3GC2VR4jTz6YatT4Ntz59/THOT7NJQhr6AyLkhhJCdkzE2cob/KouVp4ivV7Q3Fc6HX7eepHAAF/DpxwgOrg9smX6coXLgfp0b1RU2u/tUNID04rpNxTMueTtrT8WSskqvaJd3RH8r7cnRj6Y2hltkja82HlpDURDxDTRvv+krbwMr26SB/40BjpMUrDRCeKuiBahC0DCoU/4+ze1l94wVUhdkCfL0GpJrMSCDEK+XEurU18Hb7WT+ThXbkdl6VpFdHsRvqAnhR2g4b+Qzgidmuky5NUZVfEaZqV/g==
```

### 六、查看有效期

当你激活完毕后，GoLand右下角会有个Registration小长条提示框，大致的内容为：You copy is Licensed to XXX意思就会告诉你：兄弟，你已经激活成功了，激活码的许可来源是：XXX。

查看有效期的步骤为点击：Help->About这里可以看到你的GoLand的版本号、许可来源、有效期、以及一些环境

服务器激活是没有期限的，即为**永久有效**

激活码激活的有效期到**2089年**的

![](https://www.codepeople.cn/imges/go/023.png)



=====================================================================

博客地址：<https://www.codepeople.cn>

查看原文：[GoLand永久激活](http://www.imooc.com/article/details/id/288329)

=====================================================================

微信公众号：

![](https://www.codepeople.cn/imges/weixin_icon/weixin.jpg)