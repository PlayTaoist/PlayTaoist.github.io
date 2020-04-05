---
title:  "Java参数校验(validator)"
categories: validator
tags: validator
author: LIUREN
---

# Java参数校验(validator)

> Java参数校验(validator)



### 1.为什么要用validator

### javax.validation的一系列注解可以帮我们完成参数校验,免去繁琐的串行校验

### 2.什么是javax.validation

JSR303 是一套JavaBean参数校验的标准，它定义了很多常用的校验注解，我们可以直接将这些注解加在我们JavaBean的属性上面(面向注解编程的时代)，就可以在需要校验的时候进行校验了,在SpringBoot中已经包含在starter-web中,再其他项目中可以引用依赖,并自行调整版本:

```xml
    <!--jsr 303-->
        <dependency>
            <groupId>javax.validation</groupId>
            <artifactId>validation-api</artifactId>
            <version>1.1.0.Final</version>
        </dependency>
        <!-- hibernate validator-->
        <dependency>
            <groupId>org.hibernate</groupId>
            <artifactId>hibernate-validator</artifactId>
            <version>5.2.0.Final</version>
        </dependency>
```

## SpringBoot的快速框架

### 1. @Validated 声明要检查的参数

这里我们在控制器层进行注解声明

```java
 /**
     * 走参数校验注解
     *
     * @param userDTO
     * @return
     */
    @PostMapping("/save/valid")
    public RspDTO save(@RequestBody @Validated UserDTO userDTO) {
        userService.save(userDTO);
        return RspDTO.success();
    }
```

### 2. 对参数的字段进行注解标注

```java
import lombok.Data;
import org.hibernate.validator.constraints.Length;

import javax.validation.constraints.*;
import java.io.Serializable;
import java.util.Date;

/**
 * @author LiJing
 * @ClassName: UserDTO
 * @Description: 用户传输对象
 * @date 2019/7/30 13:55
 */
@Data
public class UserDTO implements Serializable {

    private static final long serialVersionUID = 1L;

    /*** 用户ID*/
    @NotNull(message = "用户id不能为空")
    private Long userId;

    /** 用户名*/
    @NotBlank(message = "用户名不能为空")
    @Length(max = 20, message = "用户名不能超过20个字符")
    @Pattern(regexp = "^[\\u4E00-\\u9FA5A-Za-z0-9\\*]*$", message = "用户昵称限制：最多20字符，包含文字、字母和数字")
    private String username;

    /** 手机号*/
    @NotBlank(message = "手机号不能为空")
    @Pattern(regexp = "^[1][3,4,5,6,7,8,9][0-9]{9}$", message = "手机号格式有误")
    private String mobile;

    /**性别*/
    private String sex;

    /** 邮箱*/
    @NotBlank(message = "联系邮箱不能为空")
    @Email(message = "邮箱格式不对")
    private String email;

    /** 密码*/
    private String password;

    /*** 创建时间 */
    @Future(message = "时间必须是将来时间")
    private Date createTime;

}
```

### 3. 在全局校验中增加校验异常

MethodArgumentNotValidException是springBoot中进行绑定参数校验时的异常,需要在springBoot中处理,其他需要处理ConstraintViolationException异常进行处理.

为了优雅一点,我们将参数异常,业务异常,统一做了一个全局异常,将控制层的异常包装到我们自定义的异常中.

为了优雅一点,我们还做了一个统一的结构体,将请求的code,和msg,data一起统一封装到结构体中,增加了代码的复用性.

```java
import com.boot.lea.mybot.dto.RspDTO;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.dao.DuplicateKeyException;
import org.springframework.web.bind.MethodArgumentNotValidException;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.RestControllerAdvice;
import org.springframework.web.servlet.NoHandlerFoundException;

import javax.validation.ConstraintViolationException;
import javax.validation.ValidationException;


/**
 * @author LiJing
 * @ClassName: GlobalExceptionHandler
 * @Description: 全局异常处理器
 * @date 2019/7/30 13:57
 */
@RestControllerAdvice
public class GlobalExceptionHandler {

    private Logger logger = LoggerFactory.getLogger(getClass());

    private static int DUPLICATE_KEY_CODE = 1001;
    private static int PARAM_FAIL_CODE = 1002;
    private static int VALIDATION_CODE = 1003;

    /**
     * 处理自定义异常
     */
    @ExceptionHandler(BizException.class)
    public RspDTO handleRRException(BizException e) {
        logger.error(e.getMessage(), e);
        return new RspDTO(e.getCode(), e.getMessage());
    }

    /**
     * 方法参数校验
     */
    @ExceptionHandler(MethodArgumentNotValidException.class)
    public RspDTO handleMethodArgumentNotValidException(MethodArgumentNotValidException e) {
        logger.error(e.getMessage(), e);
        return new RspDTO(PARAM_FAIL_CODE, e.getBindingResult().getFieldError().getDefaultMessage());
    }

    /**
     * ValidationException
     */
    @ExceptionHandler(ValidationException.class)
    public RspDTO handleValidationException(ValidationException e) {
        logger.error(e.getMessage(), e);
        return new RspDTO(VALIDATION_CODE, e.getCause().getMessage());
    }

    /**
     * ConstraintViolationException
     */
    @ExceptionHandler(ConstraintViolationException.class)
    public RspDTO handleConstraintViolationException(ConstraintViolationException e) {
        logger.error(e.getMessage(), e);
        return new RspDTO(PARAM_FAIL_CODE, e.getMessage());
    }

    @ExceptionHandler(NoHandlerFoundException.class)
    public RspDTO handlerNoFoundException(Exception e) {
        logger.error(e.getMessage(), e);
        return new RspDTO(404, "路径不存在，请检查路径是否正确");
    }

    @ExceptionHandler(DuplicateKeyException.class)
    public RspDTO handleDuplicateKeyException(DuplicateKeyException e) {
        logger.error(e.getMessage(), e);
        return new RspDTO(DUPLICATE_KEY_CODE, "数据重复，请检查后提交");
    }


    @ExceptionHandler(Exception.class)
    public RspDTO handleException(Exception e) {
        logger.error(e.getMessage(), e);
        return new RspDTO(500, "系统繁忙,请稍后再试");
    }
}
```

### 使用groups的校验

```java
import javax.validation.groups.Default;

public interface Create extends Default {
}
```

```java
import javax.validation.groups.Default;

public interface Update extends Default{
}
```

再在需要校验的地方@Validated声明校验组

```java
 /**
     * 走参数校验注解的 groups 组合校验
     *
     * @param userDTO
     * @return
     */
    @PostMapping("/update/groups")
    public RspDTO update(@RequestBody @Validated(Update.class) UserDTO userDTO) {
        userService.updateById(userDTO);
        return RspDTO.success();
    }
```

在DTO中的字段上定义好groups = {}的分组类型

```java
@Data
public class UserDTO implements Serializable {

    private static final long serialVersionUID = 1L;

    /*** 用户ID*/
    @NotNull(message = "用户id不能为空", groups = Update.class)
    private Long userId;

    /**
     * 用户名
     */
    @NotBlank(message = "用户名不能为空")
    @Length(max = 20, message = "用户名不能超过20个字符", groups = {Create.class, Update.class})
    @Pattern(regexp = "^[\\u4E00-\\u9FA5A-Za-z0-9\\*]*$", message = "用户昵称限制：最多20字符，包含文字、字母和数字")
    private String username;

    /**
     * 手机号
     */
    @NotBlank(message = "手机号不能为空")
    @Pattern(regexp = "^[1][3,4,5,6,7,8,9][0-9]{9}$", message = "手机号格式有误", groups = {Create.class, Update.class})
    private String mobile;

    /**
     * 性别
     */
    private String sex;

    /**
     * 邮箱
     */
    @NotBlank(message = "联系邮箱不能为空")
    @Email(message = "邮箱格式不对")
    private String email;

    /**
     * 密码
     */
    private String password;

    /*** 创建时间 */
    @Future(message = "时间必须是将来时间", groups = {Create.class})
    private Date createTime;

}
```

注意:在声明分组的时候尽量加上 extend javax.validation.groups.Default 否则,在你声明@Validated(Update.class)的时候,就会出现你在默认没添加groups = {}的时候的校验组@Email(message = "邮箱格式不对"),会不去校验,因为默认的校验组是groups = {Default.class}.

### restful风格用法

在多个参数校验,或者@RequestParam 形式时候,需要在controller上加注@Validated

```java
@GetMapping("/get")
    public RspDTO getUser(@RequestParam("userId") @NotNull(message = "用户id不能为空") Long userId) {
        User user = userService.selectById(userId);
        if (user == null) {
            return new RspDTO<User>().nonAbsent("用户不存在");
        }
        return new RspDTO<User>().success(user);
    }
```

```java
@RestController
@RequestMapping("user/")
@Validated
public class UserController extends AbstractController {

....省略代码...
```





=====================================================================

博客地址：<https://www.codepeople.cn>

=====================================================================

微信公众号：

![](https://www.codepeople.cn/imges/weixin_icon/weixin.jpg)