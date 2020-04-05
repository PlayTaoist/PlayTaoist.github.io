---
title:  "SpringBoot2.0针对请求参数@RequestBody验证的统一拦截"
categories: SpringBoot2.0 Shiro
tags: SpringBoot2.0 Shiro
author: LIUREN
---

# SpringBoot2.0针对请求参数@RequestBody验证统一拦截

> 针对SpringBoot2中请求参数统一进行拦截处理问题



## 第一步：建立请求实体类RequetsUserEntity.java

RequetsUserEntity.java

```java
/**
 * @Title: RequetsUserEntity.java
 * @Package io.renren.modules.wx.vo
 * @Description: 
 * Copyright: Copyright (c) 2019 www.codepeople.cn Inc. All rights reserved. 
 * Website: www.codepeople.cn 
 * @Author 刘仁
 * @DateTime 2019年4月12日 上午9:28:53
 * @version V1.0
 */

package io.renren.modules.wx.vo;

import java.io.Serializable;

import javax.validation.constraints.NotBlank;

import io.swagger.annotations.ApiModel;
import io.swagger.annotations.ApiModelProperty;
import lombok.Data;

/**
 * @ClassName: RequetsUserEntity
 * @Description: 
 * @Author 刘仁
 * @DateTime 2019年4月12日 上午9:28:53 
 */
@Data
@ApiModel(description = "RequetsUserEntity")
public class RequetsUserEntity implements Serializable{
	
	private static final long serialVersionUID = 8841143276870083920L;
	/**
	 * appid
	 */
	@ApiModelProperty(value = "appid")
	@NotBlank(message="不能为空")
	private String appid;
	/**
	 * code
	 */
	@ApiModelProperty(value = "code")
	@NotBlank(message="不能为空")
	private String code;
	/**
	 * nickName
	 */
	@ApiModelProperty(value = "微信昵称")
	@NotBlank(message="不能为空")
	private String nickName;
	/**
	 * 0未知	1男性	 2女性
	 */
	@ApiModelProperty(value = "0未知	1男性	 2女性")
	@NotBlank(message="不能为空")
	private String gender;
	/**
	 * en英文	zh_CN简体中文	zh_TW繁体中文
	 */
	@ApiModelProperty(value = "en英文	zh_CN简体中文	zh_TW繁体中文")
	private String language;
	/**
	 * 用户所在城市
	 */
	@ApiModelProperty(value = "用户所在城市")
	private String city;
	/**
	 * 用户所在省份
	 */
	@ApiModelProperty(value = "用户所在省份")
	private String province;
	/**
	 * 用户所在国家
	 */
	@ApiModelProperty(value = "用户所在国家")
	private String country;
	/**
	 * 用户头像图片的 URL
	 */
	@ApiModelProperty(value = "用户头像图片的 URL")
	private String avatarUrl;
	/**
	 * 使用 sha1( rawData + sessionkey ) 得到字符串，用于校验用户信息，详见 用户数据的签名验证和加解密
	 */
	@ApiModelProperty(value = "signature")
	@NotBlank(message="不能为空")
	private String signature;
	/**
	 * 不包括敏感信息的原始数据字符串，用于计算签名
	 */
	@ApiModelProperty(value = "rawData")
	@NotBlank(message="不能为空")
	private String rawData;
	/**
	 * 包括敏感数据在内的完整用户信息的加密数据，详见 用户数据的签名验证和加解密
	 */
	@ApiModelProperty(value = "encryptedData")
	@NotBlank(message="不能为空")
	private String encryptedData;
	/**
	 * 加密算法的初始向量，详见 用户数据的签名验证和加解密
	 */
	@ApiModelProperty(value = "iv")
	@NotBlank(message="不能为空")
	private String iv;
	/**
	 * 登录类型 1 微信医生端小程序 2微信用户端小程序 3微网站 4手机app 5微信公众号
	 */
	@ApiModelProperty(value = "1 微信医生端小程序 2微信用户端小程序 3微网站 4手机app 5微信公众号")
	@NotBlank(message="不能为空")
	private String loginType; 
	
	
}
```

## 第二步：建立Controller进行验证

LoginController.java

```java
/**
 * @Title: WxLoginController.java
 * @Package io.renren.modules.wx.controller
 * @Description: 
 * Copyright: Copyright (c) 2019 www.codepeople.cn Inc. All rights reserved. 
 * Website: www.codepeople.cn
 * @Author 刘仁
 * @DateTime 2019年4月3日 下午3:34:44
 * @version V1.0
 */

package io.renren.modules.wx.controller;

import org.springframework.validation.annotation.Validated;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import io.renren.common.utils.R;
import io.renren.modules.wx.vo.RequetsUserEntity;
import io.swagger.annotations.Api;
import io.swagger.annotations.ApiOperation;
import lombok.extern.slf4j.Slf4j;

/**
 * @ClassName: LoginController
 * @Description:
 * @Author 刘仁
 * @DateTime 2019年4月3日 下午3:34:44
 */
@Api(tags = "微信小程序授权登录接口")
@RestController
@RequestMapping("/wx/user/{appid}")
@Slf4j
public class LoginController {

	@ApiOperation("微信小程序授权登录")
	@PostMapping("/login")
	public R login(@Validated @RequestBody RequetsUserEntity req) {
		// TODO 实现自己的业务
		log.info("登录成功");
		return R.ok("登录成功");
	}

}
```



## 第三步：设置全局异常进行拦截

RRExceptionHandler.java

```java
/**
 * Copyright (c) 2016-2019 人人开源 All rights reserved.
 *
 * https://www.renren.io
 *
 * 版权所有，侵权必究！
 */

package io.renren.common.exception;

import org.apache.shiro.authz.AuthorizationException;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.dao.DuplicateKeyException;
import org.springframework.http.HttpStatus;
import org.springframework.validation.FieldError;
import org.springframework.web.bind.MethodArgumentNotValidException;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.ResponseStatus;
import org.springframework.web.bind.annotation.RestControllerAdvice;

import io.renren.common.utils.R;

/**
 * 异常处理器
 *
 * @author Mark sunlightcs@gmail.com
 */
@RestControllerAdvice
public class RRExceptionHandler {
	private Logger logger = LoggerFactory.getLogger(getClass());

	/**
	 * 处理自定义异常
	 */
	@ExceptionHandler(RRException.class)
	public R handleRRException(RRException e){
		R r = new R();
		r.put("code", e.getCode());
		r.put("msg", e.getMessage());

		return r;
	}

	@ExceptionHandler(DuplicateKeyException.class)
	public R handleDuplicateKeyException(DuplicateKeyException e){
		logger.error(e.getMessage(), e);
		return R.error("数据库中已存在该记录");
	}

	@ExceptionHandler(AuthorizationException.class)
	public R handleAuthorizationException(AuthorizationException e){
		logger.error(e.getMessage(), e);
		return R.error("没有权限，请联系管理员授权");
	}

	@ExceptionHandler(Exception.class)
	public R handleException(Exception e){
		logger.error(e.getMessage(), e);
		return R.error();
	}
	
	@ExceptionHandler(MethodArgumentNotValidException.class)
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    public R validationError(MethodArgumentNotValidException ex) {
        FieldError fieldError = ex.getBindingResult().getFieldError();
        logger.error(fieldError.getField()+fieldError.getDefaultMessage());
        return R.error(9999, fieldError.getField()+fieldError.getDefaultMessage());
    }
}
```

## 第四步：然后运行SpringBoot项目发送http请求，获取返回信息

![](https://www.codepeople.cn/imges/exception/001.png)

## 第五步：针对R.java类信息如下

R.java

```java
/**
 * Copyright (c) 2016-2019 人人开源 All rights reserved.
 *
 * https://www.renren.io
 *
 * 版权所有，侵权必究！
 */

package io.renren.common.utils;

import java.util.HashMap;
import java.util.Map;

/**
 * 返回数据
 *
 * @author Mark sunlightcs@gmail.com
 */
public class R extends HashMap<String, Object> {
	private static final long serialVersionUID = 1L;
	
	public R() {
		put("code", 0);
		put("msg", "success");
	}
	
	public static R error() {
		return error(500, "未知异常，请联系管理员");
	}
	
	public static R error(String msg) {
		return error(500, msg);
	}
	
	public static R error(int code, String msg) {
		R r = new R();
		r.put("code", code);
		r.put("msg", msg);
		return r;
	}

	public static R ok(String msg) {
		R r = new R();
		r.put("msg", msg);
		return r;
	}
	
	public static R ok(Map<String, Object> map) {
		R r = new R();
		r.putAll(map);
		return r;
	}
	
	public static R ok() {
		return new R();
	}

	@Override
	public R put(String key, Object value) {
		super.put(key, value);
		return this;
	}
}
```

==================================================================

[博客地址](https://www.codepeople.cn)：<https://www.codepeople.cn>

==================================================================

微信公众号：

![](https://www.codepeople.cn/imges/weixin_icon/weixin.jpg)