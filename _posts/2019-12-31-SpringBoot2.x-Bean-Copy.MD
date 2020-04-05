---
title:  "使用自定义的BeanUtils拷贝List数组"
categories: BeanHelper
tags: BeanHelper
author: LIUREN
---

# 使用自定义的BeanUtils拷贝List数组

> 可以copy对象也可以copy数组
>
> 作者：liuren
>
> 链接：https://www.codepeople.cn
>
> 来源：GitHub
>
> 著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。




BeanHelper.java

```java
package cn.codepeople.util;
import org.springframework.beans.BeanUtils;

import java.util.List;
import java.util.Objects;
import java.util.function.Supplier;
import java.util.stream.Collectors;


public class BeanHelper {
	/**
     * 拷贝指定源列表 到 指定目标bean类型，并返回目标bean列表
     *
     * @param targetClazz 目标bean 类型
     * @param sourceList  源bean 列表
     * @param <T>         指目标bean类型
     * @param <D>         指代源bean类型
     * @return 返回指定目标bean类型的列表
     */
    @Deprecated
    public static <T, D> List<T> copyForList(Class<T> targetClazz, List<D> sourceList) {
        if (Objects.isNull(sourceList) || Objects.isNull(targetClazz)) {
            return null;
        }
        return sourceList.stream().filter(Objects::nonNull).map(d -> {
            T t = null;
            try {
                //使用反射构建对象
                t = targetClazz.newInstance();
            } catch (InstantiationException | IllegalAccessException e) {
                e.printStackTrace();
            }
            if (Objects.nonNull(t)) {
                BeanUtils.copyProperties(d, t);
            }
            return t;
        }).filter(Objects::nonNull).collect(Collectors.toList());
    }

    /**
     * 拷贝指定源列表 到 指定目标bean类型，并返回目标bean列表
     * List<UserDto> userDtos = BeanHelper.copyForList(UserDto::new, userDos);
     * @param targetSupplier 目标bean对象提供者
     * @param sourceList     源bean 列表
     * @param <T>            指目标bean类型
     * @param <D>            指代源bean类型
     * @return 返回指定目标bean类型的列表
     */
    public static <T, D> List<T> copyForList(Supplier<T> targetSupplier, List<D> sourceList) {
        if (Objects.isNull(sourceList) || Objects.isNull(targetSupplier)) {
            return null;
        }
        return sourceList.stream().filter(Objects::nonNull).map(d ->
                BeanHelper.copyForBean(targetSupplier,d))
                .filter(Objects::nonNull).collect(Collectors.toList());
    }

    /**
     * 拷贝指定bean 到目标bean
     * 用法：
     * UserDto userDto=BeanHelper.copyForBean(UserDto::new, useDo);
     * @param targetSupplier
     * @param d
     * @param <T>
     * @param <D>
     * @return
     */
    public  static <T, D> T  copyForBean(Supplier<T> targetSupplier, D d) {
        if (Objects.isNull(targetSupplier) || Objects.isNull(d)) {
            return null;
        }
        T t = null;
        t = targetSupplier.get();
        if (Objects.nonNull(t)) {
            BeanUtils.copyProperties(d, t);
        }
        return t;
    }
}
```

如何使用，使用方法如下：

```java
package cn.codepeople.util;

import java.util.ArrayList;
import java.util.List;

import cn.codepeople.entity.User;
import cn.codepeople.entity.UserVO;

public class Test {

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		copyEntity();
		//JDK1.7一下使用如下方法
		List<UserVO> userVOListx =BeanHelper.copyForList(UserVO.class, copyEntity());
		System.out.println(userVOListx);
		System.out.println("============================================>>>");
		//JDK1.8使用该方法
		List<UserVO> userVOList =BeanHelper.copyForList(UserVO::new, copyEntity());
		System.out.println(userVOList);
	}

	private static List<User> copyEntity() {
		List<User> userList = new ArrayList<>();
		User user1 = new User();
		user1.setId(1);
		user1.setName("zhangsan1");
		user1.setPassword("123456");
		user1.setPerms("system:test:add");
		
		User user2 = new User();
		user2.setId(2);
		user2.setName("zhangsan2");
		user2.setPassword("123456");
		user2.setPerms("system:test:edit");
		
		userList.add(user1);
		userList.add(user2);
		return userList;
	}

}
```

主要注意JDK1.8和JDK1.8一下版本两种不同的调用方法

控制台打印的结果如下：

```
[UserVO(id=1, name=zhangsan1), UserVO(id=2, name=zhangsan2)]
============================================>>>
[UserVO(id=1, name=zhangsan1), UserVO(id=2, name=zhangsan2)]
```

如果类型变化会获取不到指定的值



博客地址：<https://www.codepeople.cn>

=====================================================================

微信公众号：

![](https://www.codepeople.cn/imges/weixin_icon/weixin.jpg)