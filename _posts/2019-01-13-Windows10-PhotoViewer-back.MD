---
title:  "Windows照片查看器-召回大法"
categories: windows
tags: windows
author: LIUREN
---

# Windows照片查看器-召回大法

> Win10里面依然有一个非常糟糕的应用，那就是“照片”，也就是Win10自带的图片/照片浏览应用。这个应用多年来毫无改进，依然没有查看100%比例图片的功能，不能便捷进入全屏模式，最要命的是：它经常出问题，无法浏览“下一张”照片，使得用户不得不看一张，关闭一张，再去双击下一张。
>
> 这让我们非常怀念Win7时代的“Windows照片查看器”，那是绝对经典的应用，各种方便、功能各种全面、各种稳定！但狗血的是，Win10系统下，这个功能消失了！而且你还下载不到这个应用。没关系，这里我们就教大家在Win10上寻回“Windows照片查看器”。



## 一.按下Win+R组合键，打开注册表编辑器

![006](https://www.codepeople.cn/imges/ui/005.png)



## 二.在左侧选项中依次进入

HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows Photo Viewer\Capabilities\FileAssociations

![006](https://www.codepeople.cn/imges/ui/006.png)



## 三.在界面右侧点鼠标右键，选择“新建→字符串值”

![006](https://www.codepeople.cn/imges/ui/007.png)



## 四.新建的字符串值的名称叫做“.png”、“.jpg”等。

![006](https://www.codepeople.cn/imges/ui/009.png)



## 五.双击一个新建的值，在弹出界面的“数值数据”栏中，依葫芦画瓢，填写“PhotoViewer.FileAssoc.Tiff”字样，然后记得按下“确定”按键

![006](https://www.codepeople.cn/imges/ui/008.png)

可以复制粘贴，因为每种图片格式你都要添加一次。最终结果如下

![006](https://www.codepeople.cn/imges/ui/010.png)



## 六.大功告成了！这时当你双击图片文件时，会弹出一个对话框，询问你打开方式，你选择“Windows照片查看器”（之前没有这个选项），同时勾选“始终使用此应用打开文件”选项并按下“确定”按钮。每种图片第一次打开的时候都会询问一次，你勾选了“始终使用……”后，Windows照片查看器就成为默认应用了——完美了！

![006](https://www.codepeople.cn/imges/ui/011.png)





