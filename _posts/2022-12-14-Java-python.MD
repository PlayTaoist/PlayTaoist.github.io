---
title:  "Java调用Python脚本并返回数据"
categories:  Java Python
tags: Java Python
author: LIUREN
---

# Java调用Python脚本并返回数据

> Java调用Python脚本并返回数据
>
> 协议：CC BY-SA 4.0 https://creativecommons.org/licenses/by-sa/4.0/  
>
> 版权声明：本文为原创文章，遵循 CC 4.0 BY-SA 版权协议，转载请附上原文出处链接及本声明。



[TOC]

### Python实现源码



```python
def returnStr():
    str1 = '{"cpu":{"appCupRate":9.51,"systemCpuRate": 0},"memory":{"totalPass":588.65,"nativePass":183.1,"dalvikPass":18.0},"flow":{"upFlow":0.0,"downFlow":0.0},"fps":{"fps":0,"jank":0},"battery":{"level":86,"temperature":26.6}}'
    return str1


if __name__ == '__main__':
    print(returnStr())
```

Java版本

```java
package com.p6spy.p6spy.controller.opencv;

import org.opencv.core.Core;
import org.opencv.core.Core.MinMaxLocResult;
import org.opencv.highgui.HighGui;
import org.opencv.core.CvType;
import org.opencv.core.Mat;
import org.opencv.core.Point;
import org.opencv.core.Size;
import org.opencv.imgcodecs.Imgcodecs;
import org.opencv.imgproc.Imgproc;
/***
 * @ClassName: Good-Good-Good-Good-Good-Good
 * @Description: 
 * @Author liuren
 * @DateTime 2022年8月17日 下午3:35:50
 */
public class Good {

	public static void main(String[] args) throws Exception {
        try {
            System.out.println("start");
            //环境和main
            String[] args1=new String[]{"D:\\moble_python\\project_zip\\PythonEnv\\python.exe","C:\\Users\\liuren\\Desktop\\sudy\\study\\AAA.py"};
            Process pr=Runtime.getRuntime().exec(args1);

            BufferedReader in = new BufferedReader(new InputStreamReader(
                    pr.getInputStream()));
            String line;
            while ((line = in.readLine()) != null) {
                System.out.println(line);
            }
            in.close();
            pr.waitFor();
            System.out.println("end");
        } catch (Exception e) {
            e.printStackTrace();
        };
    }

	
}
```

=====================================================================

微信公众号：

![](https://www.codepeople.cn/imges/weixin_icon/weixin.jpg)