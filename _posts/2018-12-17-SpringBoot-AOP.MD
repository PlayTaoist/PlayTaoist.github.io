---
title:  "SpringBoot的AOP切面编程问题"
categories: SpringBoot2.0
tags: SpringBoot2.0
author: LIUREN
---

# SpringBoot的AOP切面编程问题



> SpringBoot的AOP切面编程问题,使用切面编程可以用来操作日志，系统日志，以及web请求；都能够清晰的记录其访问记录，SpringBoot的切面编程是Spring的核心模块之一



## SpringBoot的AOP切面编程



#### 引入AOP依赖

在Spring Boot中引入AOP就跟引入其他模块一样，非常简单，只需要在`pom.xml`中加入如下依赖：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-aop</artifactId>
</dependency>
```

   **关于Spring AOP的一些术语**

- **切面（Aspect）**：在Spring AOP中，切面可以使用通用类或者在普通类中以@Aspect 注解（@AspectJ风格）来实现

- **连接点（Joinpoint）**：在Spring AOP中一个连接点代表一个方法的执行

- **通知（Advice）**：在切面的某个特定的连接点（Joinpoint）上执行的动作。通知有各种类型，其中包括"around"、"before”和"after"等通知。许多AOP框架，包括Spring，都是以拦截器做通知模型， 并维护一个以连接点为中心的拦截器链

- **切入点（Pointcut）**：定义出一个或一组方法，当执行这些方法时可产生通知，Spring缺省使用AspectJ切入点语法。

   **通知类型**

- **前置通知（@Before）**：在某连接点（join point）之前执行的通知，但这个通知不能阻止连接点前的执行（除非它抛出一个异常）

- **返回后通知（@AfterReturning）**：在某连接点（join point）正常完成后执行的通知：例如，一个方法没有抛出任何异常，正常返回

- **抛出异常后通知（@AfterThrowing）**：方法抛出异常退出时执行的通知

- **后通知（@After）**：当某连接点退出的时候执行的通知（不论是正常返回还是异常退出）

- **环绕通知（@Around）**：包围一个连接点（join point）的通知，如方法调用。这是最强大的一种通知类型，环绕通知可以在方法调用前后完成自定义的行为，它也会选择是否继续执行连接点或直接返回它们自己的返回值或抛出异常来结束执行

​      **@AspectJ风格的AOP配置**

​     Spring AOP配置有两种风格：

- XML风格 = 采用声明形式实现Spring AOP 
- AspectJ风格 = 采用注解形式实现Spring AOP

以上内容来源：https://tiankong6622.iteye.com/blog/2092733

我的日志记录表：

```mysql
CREATE TABLE `t_log` (  
  `id` bigint(20) unsigned NOT NULL AUTO_INCREMENT,  
  `userid` bigint(20) unsigned NOT NULL,
  `userName` bigint(20) unsigned NOT NULL,
  `createDate` datetime NOT NULL COMMENT '创建日期',  
  `content` text COMMENT '日志内容',  
  `operation` varchar(250) NOT NULL DEFAULT '' COMMENT '用户所做的操作', 
  `ip` varchar(250) NOT NULL DEFAULT '' COMMENT '登录IP地址',  
  PRIMARY KEY (`id`)  
) ENGINE=InnoDB DEFAULT CHARSET=utf8;  
```

 **日志记录POJO类：**

```
package com.hz.yiliao.orm;  
  
import java.io.Serializable;  
import java.util.Date;  
  
/** 
 * 系统日志 
 * @author Administrator 
 * 
 */  
public class Log implements Serializable{  
  
    private static final long serialVersionUID = -7372303516713218870L;  
      
    private Long id;//  
    private Long userId;// 
    private String userName;//日志内容
    private Date createDate;//创建日期  
    private String content;//日志内容  
    private String operation;//用户所做的操作  
    private String ip;//用户登录IP
      
    //get set方法省略，自己生成  
      
      
} 
```

**日志记录Mapper接口与XML配置文件：**

```xml
<?xml version="1.0" encoding="UTF-8"?>  
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">  
<mapper namespace="com.hz.yiliao.dao.LogMapper">  
    <resultMap type="com.hz.yiliao.orm.Log" id="LogMapper">  
        <id property="id" column="id"/>  
        <result property="userId" column="userid" />  
        <result property="createDate" column="createdate" />  
        <result property="content" column="content" />  
        <result property="operation" column="operation" />  
    </resultMap>  
      
    <sql id="LogSQL">  
        userid,createdate,content,operation  
    </sql>  
      
    <insert id="insert" parameterType="com.hz.yiliao.orm.Log" useGeneratedKeys="true" keyProperty="id"  keyColumn="id">  
        insert into t_log(<include refid="LogSQL"/>) value(#{userId},NOW(),#{content},#{operation})  
    </insert>  
      
</mapper>
```

**日志记录Service层:**

```java
package com.hz.yiliao.dao;  
  
import java.util.List;  
import java.util.Map;  
  
import com.hz.yiliao.orm.Log;  
  
/** 
 * 系统日志接口 
 * @author Administrator 
 * 
 */  
public interface LogMapper {  
  
    /** 
     * 新增 
     * @param log 
     */  
    public void insert(Log log);  
      
}
```

**LogMapper.xml配置文件**

```xml
<?xml version="1.0" encoding="UTF-8"?>  
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">  
<mapper namespace="com.hz.yiliao.dao.LogMapper">  
    <resultMap type="com.hz.yiliao.orm.Log" id="LogMapper">  
        <id property="id" column="id"/>  
        <result property="userId" column="userid" />  
        <result property="createDate" column="createdate" />  
        <result property="content" column="content" />  
        <result property="operation" column="operation" />  
    </resultMap>  
      
    <sql id="LogSQL">  
        userid,createdate,content,operation  
    </sql>  
      
    <insert id="insert" parameterType="com.hz.yiliao.orm.Log" useGeneratedKeys="true" keyProperty="id"  keyColumn="id">  
        insert into t_log(<include refid="LogSQL"/>) value(#{userId},NOW(),#{content},#{operation})  
    </insert>  
      
</mapper>
```

 在MyBatis配置文件mybatis-config.xml中配置POJO，根据自己的要求配置

**创建切面类LogAspect:**

```java
package com.jixi.controller;

import com.jixi.pojo.Log;
import com.jixi.pojo.User;
import com.jixi.service.ILogService;
import com.jixi.service.IUserService;
import org.aspectj.lang.JoinPoint;
import org.aspectj.lang.annotation.AfterReturning;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Pointcut;
import org.springframework.beans.factory.annotation.Autowired;

import java.lang.reflect.Method;
import java.util.Date;


@Aspect
public class LogAspect {

    public Integer id=null;

    @Autowired
    private ILogService logService;
    @Autowired
    private IUserService userService;


    /**
     * 管理员登录方法的切入点
     */
    @Pointcut("execution(* com.jixi.service.impl.*.selectByUserName(..))")
    public void loginCell(){}

    /**
     * 添加业务逻辑方法切入点
     */
    @Pointcut("execution(* com.jixi.service.impl.*.add*(..))")
    public void insertCell() {}

    /**
     * 修改业务逻辑方法切入点
     */
    @Pointcut("execution(* com.jixi.service.impl.*.update*(..))")
    public void updateCell() {}

    /**
     * 删除业务逻辑方法切入点
     */
    @Pointcut("execution(* com.jixi.service.impl.*.delete*(..))")
    public void deleteCell() {}

    /**
     * 导入业务逻辑方法切入点
     */
    @Pointcut("execution(* com.jixi.service.impl.*.import*(..))")
    public void importCell() {}

    /**
     * 导出业务逻辑方法切入点
     */
    @Pointcut("execution(* com.jixi.service.impl.*.export*(..))")
    public void exportCell() {}

    /**
     * 登录操作(后置通知)
     * @param joinPoint
     * @param object
     * @throws Throwable
     */
    @AfterReturning(value = "loginCell()", argNames = "joinPoint,object", returning = "object")
    public void loginLog(JoinPoint joinPoint, Object object) throws Throwable {
        User user=(User)object;
        if (user==null) {
            return;
        }
        if (joinPoint.getArgs() == null) {// 没有参数
            return;
        }
        id=user.getId();
        // 获取方法名
        String methodName = joinPoint.getSignature().getName();
        // 获取操作内容
        String opContent = optionContent(joinPoint.getArgs(), methodName);

        Log log = new Log();
        log.setContent(opContent);
        log.setAdminid(user.getId().toString());
        log.setAdmin(user.getUsername());
        log.setCreatedate(new Date());
        log.setOperation("登录");
        logService.saveLog(log);
    }

    /**
     * 添加操作日志(后置通知)
     *
     * @param joinPoint
     * @param object
     */
    @AfterReturning(value = "insertCell()", argNames = "joinPoint,object", returning = "object")
    public void insertLog(JoinPoint joinPoint, Object object) throws Throwable {
        // Admin admin=(Admin)
        // request.getSession().getAttribute("businessAdmin");
        // 判断参数
        if (joinPoint.getArgs() == null) {// 没有参数
            return;
        }
        // 获取方法名
        String methodName = joinPoint.getSignature().getName();
        // 获取操作内容
        String opContent = optionContent(joinPoint.getArgs(), methodName);

        Log log = new Log();
        log.setContent(opContent);
        log.setAdminid(id.toString());
        log.setAdmin(userService.selectOne(id).getUsername());
        log.setOperation("添加"+getMethodChineseName(methodName));
        log.setCreatedate(new Date());
        logService.saveLog(log);
    }

    /**
     * 管理员修改操作日志(后置通知)
     *
     * @param joinPoint
     * @param object
     * @throws Throwable
     */
    @AfterReturning(value = "updateCell()", argNames = "joinPoint,object", returning = "object")
    public void updateLog(JoinPoint joinPoint, Object object) throws Throwable {
        // Admin admin=(Admin)
        // request.getSession().getAttribute("businessAdmin");

        // 判断参数
        if (joinPoint.getArgs() == null) {// 没有参数
            return;
        }
        // 获取方法名
        String methodName = joinPoint.getSignature().getName();
        // 获取操作内容
        String opContent = optionContent(joinPoint.getArgs(), methodName);

        // 创建日志对象
        Log log = new Log();
        log.setContent(opContent);
        log.setAdminid(id.toString());
        log.setAdmin(userService.selectOne(id).getUsername());
        log.setOperation("修改"+getMethodChineseName(methodName));// 操作
        log.setCreatedate(new Date());
        logService.saveLog(log);
    }

    /**
     * 管理员导入操作日志(后置通知)
     *
     * @param joinPoint
     * @param object
     * @throws Throwable
     */
    @AfterReturning(value = "importCell()", argNames = "joinPoint,object", returning = "object")
    public void importLog(JoinPoint joinPoint, Object object) throws Throwable {
        // Admin admin=(Admin)
        // request.getSession().getAttribute("businessAdmin");

        // 判断参数
        if (joinPoint.getArgs() == null) {// 没有参数
            return;
        }
        // 获取方法名
        String methodName = joinPoint.getSignature().getName();
        // 获取操作内容
        String opContent = optionContent(joinPoint.getArgs(), methodName);

        // 创建日志对象
        Log log = new Log();
        log.setContent(opContent);
        log.setAdminid(id.toString());
        log.setAdmin(userService.selectOne(id).getUsername());
        log.setOperation("导入"+getMethodChineseName(methodName));// 操作
        log.setCreatedate(new Date());
        logService.saveLog(log);
    }

    /**
     * 管理员导出操作日志(后置通知)
     *
     * @param joinPoint
     * @param object
     * @throws Throwable
     */
    @AfterReturning(value = "exportCell()", argNames = "joinPoint,object", returning = "object")
    public void exportLog(JoinPoint joinPoint, Object object) throws Throwable {
        // Admin admin=(Admin)
        // request.getSession().getAttribute("businessAdmin");

        // 判断参数
        if (joinPoint.getArgs() == null) {// 没有参数
            return;
        }
        // 获取方法名
        String methodName = joinPoint.getSignature().getName();
        // 获取操作内容
        String opContent = optionContent(joinPoint.getArgs(), methodName);

        // 创建日志对象
        Log log = new Log();
        log.setContent(opContent);
        log.setAdminid(id.toString());
        log.setAdmin(userService.selectOne(id).getUsername());
        log.setOperation("导出"+getMethodChineseName(methodName));// 操作
        log.setCreatedate(new Date());
        logService.saveLog(log);
    }

    /**
     * 删除操作
     *
     * @param joinPoint
     * @param object
     */
    @AfterReturning(value = "deleteCell()", argNames = "joinPoint,object", returning = "object")
    public void deleteLog(JoinPoint joinPoint, Object object) throws Throwable {
        // Admin admin=(Admin)
        // request.getSession().getAttribute("businessAdmin");
        // 判断参数
        if (joinPoint.getArgs() == null) {// 没有参数
            return;
        }
        // 获取方法名
        String methodName = joinPoint.getSignature().getName();

        StringBuffer rs = new StringBuffer();
        rs.append(methodName);
        String className = null;
        for (Object info : joinPoint.getArgs()) {
            // 获取对象类型
            className = info.getClass().getName();
            className = className.substring(className.lastIndexOf(".") + 1);
            rs.append("[参数，类型:" + className + "，值:(id:"
                    + joinPoint.getArgs()[0] + ")");
        }
        rs.append("]");

        // 创建日志对象
        Log log = new Log();
        log.setContent(rs.toString());
        log.setAdminid(id.toString());
        log.setAdmin(userService.selectOne(id).getUsername());
        log.setOperation("删除"+getMethodChineseName(methodName));// 操作
        log.setCreatedate(new Date());
        logService.saveLog(log);
    }

    /**
     * 使用Java反射来获取被拦截方法(insert、update)的参数值， 将参数值拼接为操作内容
     *
     * @param args
     * @param mName
     * @return
     */
    public String optionContent(Object[] args, String mName) {
        if (args == null) {
            return null;
        }
        StringBuffer rs = new StringBuffer();
        rs.append(mName);
        String className = null;
        int index = 1;
        // 遍历参数对象
        for (Object info : args) {
            // 获取对象类型
            className = info.getClass().getName();
            className = className.substring(className.lastIndexOf(".") + 1);
            rs.append("[参数" + index + "，类型:" + className + "，值:");
            // 获取对象的所有方法
            Method[] methods = info.getClass().getDeclaredMethods();
            // 遍历方法，判断get方法
            for (Method method : methods) {
                String methodName = method.getName();
                // 判断是不是get方法
                if (methodName.indexOf("get") == -1) {// 不是get方法
                    continue;// 不处理
                }
                Object rsValue = null;
                try {
                    // 调用get方法，获取返回值
                    rsValue = method.invoke(info);
                } catch (Exception e) {
                    continue;
                }
                // 将值加入内容中
                rs.append("(" + methodName + ":" + rsValue + ")");
            }
            rs.append("]");
            index++;
        }
        return rs.toString();
    }

    /**
     * 判断操作的中文名（根据自己项目而定）
     * @param methodName
     * @return
     */
    public String getMethodChineseName(String methodName){
        if(methodName.endsWith("Allorder")){
            return "通用订单分析数据";
        }else if(methodName.endsWith("Commission")){
            return "业务员提成比例";
        }else if(methodName.endsWith("CustomerDetail")){
            return "客户月度明细数据";
        }else if(methodName.endsWith("Customer")){
            return "客户信息";
        }else if(methodName.endsWith("CustomerTypeDetail")){
            return "客户类型月度明细数据";
        }else if(methodName.endsWith("ItemBuy")){
            return "商品入库数据";
        }else if(methodName.endsWith("ItemSell")){
            return "商品出库数据";
        }else if(methodName.endsWith("Item")){
            return "商品数据";
        }else if(methodName.endsWith("Log")){
            return "系统日志数据";
        }else if(methodName.endsWith("PickorderItem")){
            return "商品售出数据";
        }else if(methodName.endsWith("ItemRank")){
            return "商品售出排行";
        }else if(methodName.endsWith("PickOrderSumAsFinance")){
            return "配货单财务数据";
        }else if(methodName.endsWith("PickOrderSum")){
            return "配货单数据";
        }else if(methodName.endsWith("SellmanDetail")){
            return "销售月度明细数据";
        }else if(methodName.endsWith("Sellman")){
            return "销售信息";
        }else if(methodName.endsWith("SellorderAll")){
            return "通用客户分析数据";
        }else if(methodName.endsWith("SellorderSum")){
            return "客户订单数据";
        }else if(methodName.endsWith("SellorderSumAsSellman")){
            return "销售的客户订单数据";
        }else if(methodName.endsWith("SellorderSumAsFinance")){
            return "客户订单财务数据";
        }else if(methodName.endsWith("SupplierDetail")){
            return "供应商月度明细数据";
        }else if(methodName.endsWith("Supplier")){
            return "供应商信息";
        }else if(methodName.endsWith("Tax")){
            return "进销项数据";
        }else if(methodName.endsWith("User")){
            return "管理员";
        }else {
            return "";
        }
    }


}
```

![img](https://www.codepeople.cn/imges/20180201130808033.png)





**参考文章**

##### Spring Aop实现用户操作日志记录：https://blog.csdn.net/weixin_39936341/article/details/79226182

##### Spring AOP 系统日志记录：https://tiankong6622.iteye.com/blog/2092733

##### Spring Boot 使用AOP统一处理Web请求日志：http://www.spring4all.com/article/258