---
title:  "Python中把PDF转成单个或多个图片"
categories: python pdf png
tags: python pdf png
author: LIUREN
---

# Python中把PDF转成单个或多个图片

> 现在有很多在线的pdf转图片工具，但是有些数据是不想联网，更不想把数据上传到别人的服务器进行转换，那么怎么办呢？正常的Java编写一个耗费时间；现在不是有Python吗，人家有库，用起来快有方便，那我们就直接拿来用就好了，多舒服；
>
> 协议：CC BY-SA 4.0 https://creativecommons.org/licenses/by-sa/4.0/  
>
> 版权声明：本文为原创文章，遵循 CC 4.0 BY-SA 版权协议，转载请附上原文出处链接及本声明。



## 一、安装Python环境

这一步不想说了，这个比较简单。记得配置一下环境变量。不然运行脚本比较费劲！

------



## 二、Python的PDF转PNG代码

注意：第一步要安装库：`pip install pymupdf`

```python
#安装库 pip install pymupdf
import os
import fitz
pdf_dir=[]

def get_file():
	docunames = os.listdir()
	for docuname in docunames:
		if os.path.splitext(docuname)[1]=='.pdf': #目录下包含.pdf的文件
			pdf_dir.append(docuname)
def conver_img():
	for pdf in pdf_dir:
		doc = fitz.open(pdf)
		#pdf_name = os.path.splitext(pdf)[0]
		pdf_name = os.path.splitext(pdf)[0]
		print("====================================")
		print("开始转换%s.PDF文档"%pdf_name)
		print("====================================")
		print("共",doc.pageCount,"页")
		for pg in range(0,doc.pageCount):
			print("\r转换为图片",pg+1,"/",doc.pageCount,end=";")
			page=doc[pg]
			rotate = int(0) #旋转角度
			# 每个尺寸的缩放系数为2，这将为我们生成分辨率提高四倍的图像
			zoom_x=2.0
			zoom_y=2.0
			print("")
			trans = fitz.Matrix(zoom_x, zoom_y).preRotate(rotate)
			pm= page.getPixmap(matrix=trans, alpha=False)
			pm.writePNG(r'./tu'+str(pdf_name)+'-'+'{:02}.png'.format(pg))
if __name__=='__main__':
	get_file()
	conver_img()
```



脚本和PDF要在同一个文件夹下。该脚本会对文件夹下的所有PDF进行转换。

=====================================================================

微信公众号：

![](https://www.codepeople.cn/imges/weixin_icon/weixin.jpg)