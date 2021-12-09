---
title:  "SpringBoot中Word转PDF"
categories:  Word PDF
tags: Word PDF
author: LIUREN
---

# SpringBoot中Word转PDF

> SpringBoot中Word转PDF
>
> 协议：CC BY-SA 4.0 https://creativecommons.org/licenses/by-sa/4.0/  
>
> 版权声明：本文为原创文章，遵循 CC 4.0 BY-SA 版权协议，转载请附上原文出处链接及本声明。



[TOC]

## SpringBoot中Word转PDF

接上一章：

[使用Word模板导出标准表Word样式文]: https://www.codepeople.cn/2021/12/08/Word-template-Export/	"使用Word模板导出标准表Word样式文"

 把导出的Word转化为PDF文档

## 第一步：引入架包

配置pom.xml

引入包

```xml
<dependency>
		    <groupId>com.aspose</groupId>
		    <artifactId>aspose-words-jdk16</artifactId>
		    <version>15.8.0</version>
		    <scope>system</scope>
		    <systemPath>${basedir}/src/main/resources/lib/aspose-words-15.8.0-jdk16.jar</systemPath>
		</dependency>
```

引入：license.xml

```xml
<License>
  <Data>
    <Products>
      <Product>Aspose.Total for Java</Product>
      <Product>Aspose.Words for Java</Product>
    </Products>
    <EditionType>Enterprise</EditionType>
    <SubscriptionExpiry>20991231</SubscriptionExpiry>
    <LicenseExpiry>20991231</LicenseExpiry>
    <SerialNumber>8bfe198c-7f0c-4ef8-8ff0-acc3237bf0d7</SerialNumber>
  </Data>
  <Signature>sNLLKGMUdF0r8O1kKilWAGdgfs2BvJb/2Xp8p5iuDVfZXmhppo+d0Ran1P9TKdjV4ABwAgKXxJ3jcQTqE/2IRfqwnPf8itN8aFZlV3TJPYeD3yWE7IT55Gz6EijUpC7aKeoohTb4w2fpox58wWoF3SNp6sK6jDfiAUGEHYJ9pjU=</Signature>
</License>
```

实现方法：

```java
public static boolean getLicense() {
        boolean result = false;
        try {
            File file = ResourceUtils.getFile("classpath:wordtemplate/license.xml");
    		FileInputStream inputStream = new FileInputStream(file);
            License aposeLic = new License();
            aposeLic.setLicense(inputStream);
            result = true;
        } catch (Exception e) {
            e.printStackTrace();
        }
        return result;
    }
	
	public static void doc2pdf(String inPath, String outPath) {
        if (!getLicense()) { // 验证License 若不验证则转化出的pdf文档会有水印产生
            return;
        }
        try {
            long old = System.currentTimeMillis();
            File file = new File(outPath); // 新建一个空白pdf文档
            FileOutputStream os = new FileOutputStream(file);
            Document doc = new Document(inPath); // Address是将要被转化的word文档
            doc.save(os, SaveFormat.PDF);// 全面支持DOC, DOCX, OOXML, RTF HTML, OpenDocument, PDF,
                                         // EPUB, XPS, SWF 相互转换
            long now = System.currentTimeMillis();
            System.out.println("共耗时：" + ((now - old) / 1000.0) + "秒"); // 转化用时
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
```

调用方法：

```java
doc2pdf("D:\\sts4.10.0_workspace\\test\\test\\src\\main\\resources\\wordtemplate\\templateAAA.docx", "D:\\sts4.10.0_workspace\\test\\test\\src\\main\\resources\\wordtemplate\\xx.pdf");
```

word转PDF完毕，生成PDF

aspose-words-15.8.0.zip

[所需架包下载：](https://www.codepeople.cn/imges/aspose-words-15.8.0.zip) https://www.codepeople.cn/imges/aspose-words-15.8.0.zip



=====================================================================

微信公众号：

![](https://www.codepeople.cn/imges/weixin_icon/weixin.jpg)