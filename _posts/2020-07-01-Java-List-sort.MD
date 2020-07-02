---
title:  "Java中使用List<T>进行排序"
categories: Java
tags: Java List<T>
author: LIUREN
---

# Java中使用List<T>进行排序

> 有时候我们经常会用到排序功能，特别是对List进行排序。简单的List排序比较方便。但是有时候我们需要对List<T>这种对象数据进行排序。想要按照降序排列比较麻烦。网上找到的现成的解决方法，在这里记录一下方便以后查找使用，不用老百度。
>
> 协议：CC BY-SA 4.0 https://creativecommons.org/licenses/by-sa/4.0/  
>
> 版权声明：本文为原创文章，遵循 CC 4.0 BY-SA 版权协议，转载请附上原文出处链接及本声明。



## 第一步：

1、先来个简单的，上代码

```java
import java.util.ArrayList;
import java.util.Collections;
import java.util.List;
public class sort {
　　public static void main(String[] args) {
　　List<Integer> list = new ArrayList<Integer>();
　　list.add(new Integer(5));
　　list.add(new Integer(13));
　　list.add(new Integer(4));
　　list.add(new Integer(9));
　　Collections.sort(list);
　　System.out.println(list.toString());
　　}
}
```
程序运行结果：
```java
[4, 5, 9, 13]，这种简单的排序直接按照自然顺序进行升序排列。
```

2、list参数类型相对复杂的排序

```java
import java.util.ArrayList;
import java.util.Collections;
import java.util.List;
public class sort {
　　public static void main(String[] args) {
　　List<User> list = new ArrayList<User>();
　　list.add(new User("张三", 5));
　　list.add(new User("李四", 30));
　　list.add(new User("王五", 19));
　　list.add(new User("陈十七", 17)); // 陈十七永远十七岁
　　Collections.sort(list); // 按年龄排序
　　System.out.println(list.toString());
　　}
}
```



```java
public class User implements Comparable<User>{

　　private String name; //姓名

　　private int age; // 年龄


　　public User(String name, int age) {
　　　　this.name = name;
　　　　this.age = age;
　　}

　　// getter && setter
　　public String getName() {
　　　　return name;
　　}
　　public void setName(String name) {
　　　　this.name = name;
　　}
　　public int getAge() {
　　　　return age;
　　}
　　public void setAge(int age) {
　　this.age = age;
　　}
　　@Override
　　public String toString() {
　　　　return "User [name=" + name + ", age=" + age + "]";
　　}

　　@Override
　　public int compareTo(User user) {           //重写Comparable接口的compareTo方法，
　　　　return this.age - user.getAge();　　// 根据年龄升序排列，降序修改相减顺序即可
　　}
}
```

程序运行结果：

[User [name=张三, age=5], User [name=陈十七, age=17], User [name=王五, age=19], User [name=李四, age=30]]



博客地址：<https://www.codepeople.cn>

=====================================================================

微信公众号：

![](https://www.codepeople.cn/imges/weixin_icon/weixin.jpg)