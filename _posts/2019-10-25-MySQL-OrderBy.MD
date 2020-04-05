---
title:  "MySQL排序-如何根据汉字首字母排序"
categories: MySQL
tags: MySQL
author: LIUREN
---

# MySQL排序-如何根据汉字首字母排序

> Mysql排序-如何根据汉字首字母排序；MySQL排序-如何根据汉字首字母排序



#### 一、 问题

需要进行中文排序。比如说微信通讯录里面的排序列表，啊XX    排到   曾XX    等等。

#### 二、原理

如果存储汉字的字段编码使用的是GBK字符集，因为GBK内码编码时本身就采用了拼音排序的方法（常用一级汉字3755个采用拼音排序，二级汉字就不是了，但考虑到人名等都是常用汉字，因此只是针对一级汉字能正确排序也够用了），直接在查询语句后面添加ORDER BY name ASC，查询结果将按照姓氏的升序排序；

如果存储姓名的字段采用的是utf8字符集，需要在排序的时候对字段进行转码，对应的代码是ORDER BY convert(name using gbk) ASC

#### 三、解决方案



目前暂时提供三种方案，如果大神有其他的方案，可以在评论中提出，共同进步：

##### 方式一

```shell
select name from tablename WHERE 1=1 order by CONVERT( name USING gbk ) COLLATE gbk_chinese_ci ASC
```



```
SELECT	*FROM	petWHERE	1 = 1ORDER BY	CONVERT (owner USING gbk) COLLATE gbk_chinese_ci ASC
```



首先，对name字段进行gbk编码，然后，对编码后的内容根据gbk_chinese_ci进行整理排序。这样得到的结果，英文是排在中文前面的，而且是根据拼音排序的。

pet：数据库表名
owner：排序字段名
CONVERT：提供一个在不同字符集之间转换数据的方法。
COLLATE：COLLATE是一个算法语句，主要用于对字符进行排序，经常出现在表的创建语句中。sql语句里面的COLLATE主要用于对字符进行排序。

##### 方式二

```shell
SELECT *,ELT(INTERVAL (
			CONV(HEX(LEFT (CONVERT (`name` USING gbk),1)),16,10),
			0xB0A1,
			0xB0C5,
			0xB2C1,
			0xB4EE,
			0xB6EA,
			0xB7A2,
			0xB8C1,
			0xB9FE,
			0xBBF7,
			0xBFA6,
			0xC0AC,
			0xC2E8,
			0xC4C3,
			0xC5B6,
			0xC5BE,
			0xC6DA,
			0xC8BB,
			0xC8F6,
			0xCBFA,
			0xCDDA,
			0xCEF4,
			0xD1B9,
			0xD4D1),
		'A', 'B',
		'C', 'D',
		'E', 'F',
		'G', 'H',
		'J', 'K',
		'L ', 'M',
		'N ', 'O',
		'P ', 'Q',
		'R ', 'S',
		'T ', 'W',
		'X ', 'Y',
		'Z ' ) AS PY
FROM pet
ORDER BY name ASC
```

name，是属性名

pet，表名

##### 方式三

可以通过  pinyin4j 进行程序转换排序。

pinyin4j [下载](http://pinyin4j.sourceforge.net/)

四、测试案例



创建测试数据

```shell
/*
Navicat MySQL Data Transfer
 
Source Server         : localhost
Source Server Version : 50096
Source Host           : localhost:3306
Source Database       : test1
 
Target Server Type    : MYSQL
Target Server Version : 50096
File Encoding         : 65001
 
Date: 2017-02-28 16:24:14
*/
 
SET FOREIGN_KEY_CHECKS=0;
 
-- ----------------------------
-- Table structure for pet
-- ----------------------------
DROP TABLE IF EXISTS `pet`;
CREATE TABLE `pet` (
  `name` varchar(20) default NULL,
  `owner` varchar(20) default NULL,
  `species` varchar(20) default NULL,
  `sex` char(1) default NULL,
  `birth` date default NULL,
  `death` date default NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
 
-- ----------------------------
-- Records of pet
-- ----------------------------
INSERT INTO `pet` VALUES ('毛茸茸的', '哈罗德', 'cat', 'f', '1993-02-04', null);
INSERT INTO `pet` VALUES ('爪子', '李彦宏', 'cat', 'm', '1994-03-17', null);
INSERT INTO `pet` VALUES ('淡黄色的', 'Harold', 'dog', 'f', '1989-05-13', null);
INSERT INTO `pet` VALUES ('Fang', 'Benny', 'dog', 'm', '1990-08-27', null);
INSERT INTO `pet` VALUES ('重庆', 'Diane', 'dog', 'm', '1979-08-31', null);
INSERT INTO `pet` VALUES ('Chirpy', 'Gwen', 'bird', 'f', '1998-09-11', null);
```

查询语句

单个字段

```shell
SELECT NAME 
FROM
	pet 
WHERE
	1 = 1 
ORDER BY
	CONVERT ( NAME USING gbk ) COLLATE gbk_chinese_ci ASC
```

多个字段

```shell
SELECT
	* 
FROM
	pet 
WHERE
	1 = 1 
ORDER BY
	CONVERT ( NAME USING gbk ) COLLATE gbk_chinese_ci,
	CONVERT ( OWNER USING gbk ) COLLATE gbk_chinese_ci ASC
```

其他类型排序

```shell
SELECT *,ELT(INTERVAL (
			CONV(HEX(LEFT (CONVERT (`name` USING gbk),1)),16,10),
			0xB0A1,
			0xB0C5,
			0xB2C1,
			0xB4EE,
			0xB6EA,
			0xB7A2,
			0xB8C1,
			0xB9FE,
			0xBBF7,
			0xBFA6,
			0xC0AC,
			0xC2E8,
			0xC4C3,
			0xC5B6,
			0xC5BE,
			0xC6DA,
			0xC8BB,
			0xC8F6,
			0xCBFA,
			0xCDDA,
			0xCEF4,
			0xD1B9,
			0xD4D1),
		'A', 'B',
		'C', 'D',
		'E', 'F',
		'G', 'H',
		'J', 'K',
		'L ', 'M',
		'N ', 'O',
		'P ', 'Q',
		'R ', 'S',
		'T ', 'W',
		'X ', 'Y',
		'Z ' ) AS PY
FROM pet
ORDER BY name ASC
```

以上的方案还有一个小bug，多音字的问题



原文链接：<http://www.souvc.com/?p=2439>

=====================================================================

博客地址：<https://www.codepeople.cn>

=====================================================================

微信公众号：

![](https://www.codepeople.cn/imges/weixin_icon/weixin.jpg)