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

如果是JPG或者PNG转换成PDF，脚本如下：

记得根据提示添加对应模块

```py
#coding=utf-8
#!/usr/bin/env python
"""
convert image to pdf file
"""
#Author: mrbeann 
import os
import sys
import glob
import platform
from reportlab.lib.pagesizes import letter, A4, landscape
from reportlab.platypus import SimpleDocTemplate, Image
from reportlab.lib.units import inch
from reportlab.pdfgen import canvas
from reportlab import rl_settings
from PIL import Image
import importlib,sys
#importlib.reload(sys)
#sys.setdefaultencoding("utf-8")
def topdf(path,recursion=None,pictureType=None,sizeMode=None,width=None,height=None,fit=None,save=None):
  """
  Parameters
  ----------
  path : string
      path of the pictures
  recursion : boolean
        None or False for no recursion
        True for recursion to children folder
        wether to recursion or not
  pictureType : list
         type of pictures,for example :jpg,png...
  sizeMode : int
      None or 0 for pdf's pagesize is the biggest of all the pictures
      1 for pdf's pagesize is the min of all the pictures
      2 for pdf's pagesize is the given value of width and height
      to choose how to determine the size of pdf
  width : int
      width of the pdf page
  height : int
      height of the pdf page
  fit : boolean
      None or False for fit the picture size to pagesize
      True for keep the size of the pictures
      wether to keep the picture size or not
  save : string
      path to save the pdf
  """
  if platform.system() == 'Windows':
    path = path.replace('\\','/')
  if path[-1] != '/':
    path = (path + '/')
  if recursion == True:
    for i in os.listdir(path):
      if os.path.isdir(os.path.abspath(os.path.join(path, i))):
        topdf(path+i,recursion,pictureType,sizeMode,width,height,fit,save)
  filelist = []
  if pictureType == None:
    filelist = glob.glob(os.path.join(path, '*.jpg'))
  else:
    for i in pictureType:
      filelist.extend(glob.glob(os.path.join(path, '*.'+i)))
  maxw = 0
  maxh = 0
  if sizeMode == None or sizeMode == 0:
    for i in filelist:
      im = Image.open(i)
      if maxw < im.size[0]:
        maxw = im.size[0]
      if maxh < im.size[1]:
        maxh = im.size[1]
  elif sizeMode == 1:
    maxw = 999999
    maxh = 999999
    for i in filelist:
      im = Image.open(i)
      if maxw > im.size[0]:
        maxw = im.size[0]
      if maxh > im.size[1]:
        maxh = im.size[1]
  else:
    if width == None or height == None:
      raise Exception("no width or height provid")
    maxw = width
    maxh = height
  maxsize = (maxw,maxh)
  if save == None:
    filename_pdf = path + path.split('/')[-2]
  else:
    filename_pdf = save + path.split('/')[-2]
  filename_pdf = filename_pdf + '.pdf'
  c = canvas.Canvas(filename_pdf, pagesize=maxsize )
  l = len(filelist)
  for i in range(l):
    (w, h) =maxsize
    width, height = letter
    if fit == True:
      c.drawImage(filelist[i] , 0,0)
    else:
      c.drawImage(filelist[i] , 0,0,maxw,maxh)
    c.showPage()
  c.save()
def main():
	##路径根据实际情况自己改
  topdf(u'D:\Python_dll\png2pdf',pictureType=['png','jpg'],save='D:\Python_dll\png2pdf\\')
if __name__ == '__main__':
  main()
```



微信公众号：

![](https://www.codepeople.cn/imges/weixin_icon/weixin.jpg)