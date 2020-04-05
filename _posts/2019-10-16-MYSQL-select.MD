---
title:  "MySQL函数find_in_set介绍"
categories: MySQL
tags: MySQL
author: LIUREN
---

# MySQL函数find_in_set介绍

> 数据库中的某个字段我十以字符存储的，同时又以","隔开的。如果想要查询这个字段中包含某个字符串该怎么查询？使用like？感觉不妥，如果使用like匹配可能会出问题比如“15”、“152”你要查询15只要包含15的都查询出来了，这不是我想要的。如果使用in()查询，不行不能够查询出需要匹配的值。百度了一下发现了find_in_set的方法。惊喜若狂，正是我需要的查询方法有增加了一个技能。



# 场景介绍

人有时会身兼数职，需要查找出其中担任某一职务的都有哪些人，如下面position字段，不同的职务用数字表示，多个职务以逗号隔开。

![](https://www.codepeople.cn/imges/MySQL/015.png)

先要查找出担任1职务的人员，通过以下两种方式来查询。

## 方式一

采用模糊查询，匹配出1职务的记录，如下SQL：

```shell
select * from user where position like '%1%'
```

查询结果如下，仔细观察你会发现position为10的也被查出来了，但这个不符合业务要求。

![](https://www.codepeople.cn/imges/MySQL/016.png)

## 方式二

采用in的记录，如下SQL【in查询相当于多个or条件的叠加】：

```shell
select * from user where position in (1)
```



## 方式三

采用的MySQL的的原生函数FIND_IN_SET（STR，数组）来查询，SQL如下：

```shell
select * from user where find_in_set(1,position)
```

查询结果如下，符合要求。

![](https://www.codepeople.cn/imges/MySQL/017.png)

# 函数介绍

FIND_IN_SET（STR，strlist），注意其中strlist只识别英文逗号。

![](https://www.codepeople.cn/imges/MySQL/018.png)

<https://dev.mysql.com/doc/refman/5.5/en/string-functions.html#function_find-in-set>

 like是广泛的模糊匹配，字符串中没有分隔符
​    IN 后面是变量，且是模糊查询
​    Find_IN_SET 是精确匹配，字段值必须以英文”,”分隔，Find_IN_SET查询的结果要小于like查询的结果 

## 根据上面示例总结：

1. FIND_IN_SET（arg1,arg2）中arg1代表我们要查找的字符串，arg2代表中我们要匹配的列名。

2. FIND_IN_SET（arg1,arg2）中arg2中数据必须是要用 , 分割。

  　　3. FIND_IN_SET（arg1,arg2）中arg1是跟arg2中以,分割进行全匹配的。



注意在hibernate中如果要使用需要按如下写【后面要加">0"】，不然会报错

```java
@Override
	public VideoType findVideoTypeRealId(String id) {
		List parameters = new ArrayList();
		String where = " find_in_set(?,REALID)>0";
		parameters.add(id);
		String sql = "";
		return (VideoType)super.getObjectUnique(VideoType.class, null, where, parameters.toArray());
	}
```



原文链接：<https://blog.csdn.net/xueshao110/article/details/80606960>

=====================================================================

博客地址：<https://www.codepeople.cn>

=====================================================================

微信公众号：

![](https://www.codepeople.cn/imges/weixin_icon/weixin.jpg)