---
title:  "GitHub个人博客支持HTTPS的配置"
categories: GitHub HTTPS
tags: GitHub HTTPS
author: LIUREN
---

# GitHub个人博客支持HTTPS的配置

> Github 自己已经支持了 [https](https://cloud.tencent.com/developer/information/https) 了,然后准备开始操作一波试一试，过程很曲折，很少有提这方便的问题，自己正在解决这个问题，所以自己记录一下解决方案
>



## 第一步：打开 Github Page 对应的 repo，找到 setting

然后往下翻，找到了 Github Page 选项。然后选中Enforce HTTPS保存

![001](https://www.codepeople.cn/imges/github/001.png)

然后你会发现点击不了，点击不了，点击不了。

把 Custom domain 的域名删除，点击save

把刚刚删除的域名重新填进去 Custom domain，点击save

这个时候你会发现，还是无法启用，不过后面有一行小字

`Unavailableforyour site because your domain is not properly configured to support HTTPS`

什么鬼？说配置不支持HTTPS，懵逼！然后通过大量的查询资料发现需要修改域名解析地址



## 第二步：修改你的域名解析A记录，修改为以下任一个

根据GitHub提供的帮助信息，意思是你要去你的域名解析的地方修改你的解析地址

![001](https://www.codepeople.cn/imges/github/002.png)

好啊，为了HTTPS那咱就去修改解析地址，我的域名注册在`阿里云`所以就到阿里云的控制后台找到域名

![001](https://www.codepeople.cn/imges/github/003.png)

![001](https://www.codepeople.cn/imges/github/004.png)

把GitHub提供的都添加进去，然后等待10分钟左右

![001](https://www.codepeople.cn/imges/github/005.png)

## 第三步：再次你的GitHub上执行  第一步 然后等待

等它生效之后，再来一次 删除-重填 Custom domain

这个时候，会发现后面的一小行小字变成了

`Not yet available for your site because the certificate has not finished being issued.`

当然了，这个时候还是 不能启用的。

等了一段时间（我忘记我等了多久，一个小时以内吧,一个小时还是不行就多等等，还是不行就等第二天试试）

就可以启用了。

启用之后你就能看到

![001](https://www.codepeople.cn/imges/github/006.png)



然后就可以打开自己的网址<https://www.codepeople.cn> 看到浏览器上支持https的协议了。

![001](https://www.codepeople.cn/imges/github/007.png)

启用完成之后，直接访问博客 https 域名

