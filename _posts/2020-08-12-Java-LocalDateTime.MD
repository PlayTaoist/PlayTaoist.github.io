---
title:  "CentOS7.6 更新DNS，并和互联网同步时间"
categories: CentOS DNS
tags: CentOS DNS
author: LIUREN
---

# LocalDateTime的JDK1.8的新特新

> 从Java 8开始，`java.time`包提供了新的日期和时间API，主要涉及的类型有：
>
> - 本地日期和时间：`LocalDateTime`，`LocalDate`，`LocalTime`；
> - 带时区的日期和时间：`ZonedDateTime`；
> - 时刻：`Instant`；
> - 时区：`ZoneId`，`ZoneOffset`；
> - 时间间隔：`Duration`。
>
> 以及一套新的用于取代`SimpleDateFormat`的格式化类型`DateTimeFormatter`。
>
> 协议：CC BY-SA 4.0 https://creativecommons.org/licenses/by-sa/4.0/  
>
> 版权声明：本文为原创文章，遵循 CC 4.0 BY-SA 版权协议，转载请附上原文出处链接及本声明。



# 是否在时间段中

写了两种实现，date和localdatetime的两种方式

```java
public static boolean inTime(Date nowTime, Date beginTime, Date endTime) {
        Calendar date = Calendar.getInstance();
        date.setTime(nowTime);
 
        Calendar begin = Calendar.getInstance();
        begin.setTime(beginTime);
 
        Calendar end = Calendar.getInstance();
        end.setTime(endTime);
 
        if (date.after(begin) && date.before(end)) {
            return true;
        } else {
            return false;
        }
    }
 
    public static boolean hourInTime(Date beginTime, Date endTime) {
        SimpleDateFormat df = new SimpleDateFormat("HH:mm");
        Date nowTime = null;
        try {
            nowTime = df.parse(df.format(new Date()));
        } catch (ParseException e) {
            e.printStackTrace();
        }
        return inTime(nowTime, beginTime, endTime);
    }
 
    public static boolean inTime(LocalDateTime time, LocalDateTime beginTime, LocalDateTime endTime) {
        return (time.isAfter(beginTime) && time.isBefore(endTime));
    }
 
    public static boolean hourInTime(LocalDateTime beginTime, LocalDateTime endTime) {
        return inTime(LocalDateTime.now(), beginTime, endTime);
    }
```

2、格式化时间

格式化时间：

```shell
LocalDateTime time = LocalDateTime.now();
DateTimeFormatter dtf = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");
```





博客地址：<https://www.codepeople.cn>

=====================================================================

微信公众号：

![](https://www.codepeople.cn/imges/weixin_icon/weixin.jpg)