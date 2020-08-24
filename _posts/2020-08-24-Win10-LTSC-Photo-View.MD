---
title:  "恢复win10 LTSC 2019 图片查看器功能"
categories: win10
tags: win10
author: LIUREN
---

# 恢复win10 LTSC 2019 图片查看器功能

> win10 LTSC 2019 是微软的长期支持版本这个版本更新代码比较少系统比较稳定：
>
> win10的图片查看功能被屏蔽了，想要重新打开图片查看功能需要按照如下步骤去操作文件
>
> 协议：CC BY-SA 4.0 https://creativecommons.org/licenses/by-sa/4.0/  
>
> 版权声明：本文为原创文章，遵循 CC 4.0 BY-SA 版权协议，转载请附上原文出处链接及本声明。



# 昨天新装了win10 LTSC 2019，发现默认打开图片的程序是“画图”。图片查看器找不到了。可以使用下面的方法恢复图片查看器

1、开始–运行–输入“regedit”打开注册表。

2、 在打开的注册表编辑器中，从左侧依次展开：HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows Photo Viewer\Capabilities\FileAssociations

3、 双击右边的“.tif”或者“tiff”，打开“编辑字符串”对话框，我们复制其中的“数值数据”。

4、 在右侧空白地方右击，选择“新建”->“字符串值”。

5、 将名称改成你要的图片后缀，如“.jpg”， 然后双击这个新建的字符串值，粘贴“数值数据”并“确定”即可。

6、 类似的，我们再依次新建 .jpeg、.bmp、.png等图片字符串值并都设置“数值数据”为“PhotoViewer.FileAssoc.Tiff”。

7、 双击图片，弹出打开方式选择界面，我们选择“Windows照片查看程序”，并勾选下面的“始终使用此应用打开.jpg文件”，再“确定”即可。



博客地址：<https://www.codepeople.cn>

=====================================================================

微信公众号：

![](https://www.codepeople.cn/imges/weixin_icon/weixin.jpg)