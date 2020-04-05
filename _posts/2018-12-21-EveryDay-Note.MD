---
title:  "每天注意问题集锦"
categories: Note
tags: Note
author: LIUREN
---

# 每天注意问题集锦

> > 随便注意问题

| 集合类         | key          | value        | super      |
| :------------- | ------------ | ------------ | ---------- |
| HashTable      | 不允许为null | 不允许为null | Dictionary |
| CurrentHashMap | 不允许为null | 不允许为null | AbstratMap |
| TreeMap        | 不允许为null | 允许为null   | AbstratMap |
| HashMap        | 允许为null   | 允许         | AbstratMap |

**注意：** HashTable 线程是安全的，CurrentHashMap锁分段技术，TreeMap是线程不安全的，HashMap线程不安全



> mysq数据库选择字段是否遵循以下已知道的规则

| 对象     | 年龄区间   | 类型     | 字节 | 表示范围                       |
| -------- | ---------- | -------- | ---- | ------------------------------ |
| 人       | 150岁之间  | tinyint  | 1    | 无符号值：0 到 255             |
| 龟       | 数百岁     | smallint | 2    | 无符号值：0 到 65535           |
| 恐龙化石 | 数千万年   | int      | 4    | 无符号值：0 到约 42.9 亿       |
| 太阳     | 约 50 亿年 | bigint   | 8    | 无符号值：0 到约 10 的 19 次方 |

>  建立索引的时候，首先查看数据库版本，查询语句`select version();`

+ 如果数据库版本是5.6那么需要建立索引的字段的长度不能大于：

  字符集为 utf8mb4, 所以可申请索引的 varchar 长度为：767/4 ≈ 191；

  而字符集为 utf8 时，可设置的创建索引 varchar 长度为: 767/3 ≈ 255；

+ 如果数据库版本高于5.6那么需要建立索引的字段的长度不能大于：

  字符集为 utf8mb4 时，限制 varchar 字符为：3072/4 = 768;

  字符集为 utf8 时，限制 varchar 字符为：3072/3 = 1024;

> **数字型所占用的字节数如下，根据字节数即可算出表示的范围了**

TINYINT                                    1 字节 
SMALLINT                                 2 个字节 
MEDIUMINT                              3 个字节 
INT                                           4 个字节 
INTEGER                                   4 个字节 
BIGINT                                      8 个字节 
FLOAT(X)                                  4 如果 X < = 24 或 8 如果 25 < = X < = 53 
FLOAT                                       4 个字节 
DOUBLE                                    8 个字节 
DOUBLE PRECISION                  8 个字节 
REAL                                         8 个字节 
DECIMAL(M,D)                          M字节(D+2 , 如果M < D) 
NUMERIC(M,D)                          M字节(D+2 , 如果M < D)
**日期和时间类型**
列类型                                     需要的存储量 
DATE                                        3 个字节 
DATETIME                                 8 个字节 
TIMESTAMP                               4 个字节 
TIME                                         3 个字节 
YEAR                                         1 字节
**字符串类型**
列类型                                                需要的存储量 
CHAR(M)                                        M字节，1 <= M <= 255 
VARCHAR(M)                                 L+1 字节, 在此L <= M和1 <= M <= 255 
TINYBLOB, TINYTEXT                     L+1 字节, 在此L< 2 ^ 8 
BLOB, TEXT                                   L+2 字节, 在此L< 2 ^ 16 
MEDIUMBLOB, MEDIUMTEXT         L+3 字节, 在此L< 2 ^ 24 
LONGBLOB, LONGTEXT                 L+4 字节, 在此L< 2 ^ 32 
ENUM('value1','value2',...)                1 或 2 个字节, 取决于枚举值的数目(最大值65535） 
SET('value1','value2',...)                    1，2，3，4或8个字节, 取决于集合成员的数量(最多64个成员





> 使用 ISNULL()来判断是否为 NULL 值。



> 在代码中写分页查询逻辑时，若 count 为 0 应直接返回，避免执行后面的分页语句。

> 在表查询中，一律不要使用 * 作为查询的字段列表，需要哪些字段必须明确写明



+ DO（Data Object）：与数据库表结构一一对应，通过 DAO 层向上传输数据源对象。
+  DTO（Data Transfer Object）：数据传输对象，Service 或 Manager 向外传输的对象。
+ BO（Business Object）：业务对象。由 Service 层输出的封装业务逻辑的对象。
+ AO（Application Object）：应用对象。在 Web 层与 Service 层之间抽象的复用对象模型，
  极为贴近展示层，复用度不高。
+ VO（View Object）：显示层对象，通常是 Web 向模板渲染引擎层传输的对象



> A) Service/DAO 层方法命名规约
> 1） 获取单个对象的方法用 get 做前缀。
> 2） 获取多个对象的方法用 list 做前缀。
> 3） 获取统计值的方法用 count 做前缀。
> 4） 插入的方法用 save/insert 做前缀。
> 5） 删除的方法用 remove/delete 做前缀。
> 6） 修改的方法用 update 做前缀。
> B) 领域模型命名规约
> 1） 数据对象：xxxDO，xxx 即为数据表名。
> 2） 数据传输对象：xxxDTO，xxx 为业务领域相关的名称。
> 3） 展示对象：xxxVO，xxx 一般为网页名称。
> 4） POJO 是 DO/DTO/BO/VO 的统称，禁止命名成 xxxPOJO。





> 时间格式化功能SimpleDateFormat不安全换成JDK1.8的

```java
package cn.codepeople.foo;

import java.time.Instant;
import java.time.LocalDate;
import java.time.LocalDateTime;
import java.time.LocalTime;
import java.time.format.DateTimeFormatter;

public class DateTimeTest {

    public static void main(String[] args) {
        System.out.println(getTime());

    }

    public static String getTime() {
        Instant now = Instant.now();
        LocalDate lNow = LocalDate.now();
        LocalTime sNow = LocalTime.now();
        LocalDateTime dNow = LocalDateTime.now();
        DateTimeFormatter dateTimeFormatter = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");
        String nowFormat = dateTimeFormatter.format(dNow);
        System.out.println("========请高手指教======>>>>" + now);

        System.out.println("=========只有年月日======>>>>" + lNow);

        System.out.println("=========只有时分秒======>>>>" + sNow);

        System.out.println("========年月日时分秒都有======>>>>" + dNow);
        return nowFormat;
    }

}
```

