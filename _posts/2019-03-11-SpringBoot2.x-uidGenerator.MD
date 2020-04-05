---
title:  "人人网框架导入uidGenerator的ID生成方式"
categories: SpringBoot2.0 uidGenerator
tags: SpringBoot2.0 uidGenerator
author: LIUREN
---

# 人人网框架导入uidGenerator的ID生成方式

> 人人网框架导入uidGenerator的ID生成方式



## 安装环境

```shell
开发工具：Eclipse
Maven版本：apache-maven-3.5.2
java jdk 1.8
MySQL版本：5.7
```



## 一.在renren-api

创建一个节点表worker_node，创建语句：

```mysql
CREATE TABLE `worker_node` (
  `ID` bigint(20) NOT NULL AUTO_INCREMENT COMMENT 'auto increment id',
  `HOST_NAME` varchar(64) NOT NULL COMMENT 'host name',
  `PORT` varchar(64) NOT NULL COMMENT 'port',
  `TYPE` int(11) NOT NULL COMMENT 'node type: ACTUAL or CONTAINER',
  `LAUNCH_DATE` date NOT NULL COMMENT 'launch date',
  `MODIFIED` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT 'modified time',
  `CREATED` timestamp NOT NULL DEFAULT '0000-00-00 00:00:00' COMMENT 'created time',
  PRIMARY KEY (`ID`)
) ENGINE=InnoDB AUTO_INCREMENT=10 DEFAULT CHARSET=utf8 COMMENT='DB WorkerID Assigner for UID Generator';
```



## 二.在`renren-security`项目中的`pom.xml`中添加相应的架包

pom.xml添加架包内容如下：

```xml
<!--必须放在最后-->
        <dependency>
            <groupId>cn.codesheep</groupId>
            <artifactId>uid-generator</artifactId>
            <version>1.0</version>
        </dependency>
```



## 三.在`renren-api`的`io.renren.config`的文件中增加文件

CachedUidGeneratorConfig

```java
/**   
 * @author lr
 * @date 2019年3月8日 下午4:03:15 
 * @version V1.0.0   
 */
package io.renren.config;

import com.baidu.fsg.uid.impl.CachedUidGenerator;

import io.renren.service.DisposableWorkerIdAssigner;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
 
/**
 * https://github.com/baidu/uid-generator/blob/master/README.zh_cn.md
 * 百度id生成器
 * 两种生成器: DefaultUidGenerator、CachedUidGenerator。如对UID生成性能有要求, 请使用CachedUidGenerator
 * 对应Spring配置分别为: default-uid-spring.xml、cached-uid-spring.xml
 */
@Configuration
public class CachedUidGeneratorConfig {
 
    /**
     * 用完即弃的WorkerIdAssigner, 依赖DB操作
     * @return
     */
    @Bean
    public DisposableWorkerIdAssigner disposableWorkerIdAssigner(){
        return new DisposableWorkerIdAssigner();
    }
 
    @Bean
    public CachedUidGenerator cachedUidGenerator(DisposableWorkerIdAssigner disposableWorkerIdAssigner){
        CachedUidGenerator cachedUidGenerator = new CachedUidGenerator();
        cachedUidGenerator.setWorkerIdAssigner(disposableWorkerIdAssigner);
        //以下为可选配置, 如未指定将采用默认值
        cachedUidGenerator.setTimeBits(29);
        cachedUidGenerator.setWorkerBits(21);
        cachedUidGenerator.setSeqBits(13);
        cachedUidGenerator.setEpochStr("2016-09-20");
 
        //RingBuffer size扩容参数, 可提高UID生成的吞吐量
        //默认:3， 原bufferSize=8192, 扩容后bufferSize= 8192 << 3 = 65536
        cachedUidGenerator.setBoostPower(3);
        // 指定何时向RingBuffer中填充UID, 取值为百分比(0, 100), 默认为50
        // 举例: bufferSize=1024, paddingFactor=50 -> threshold=1024 * 50 / 100 = 512.
        // 当环上可用UID数量 < 512时, 将自动对RingBuffer进行填充补全
        //<property name="paddingFactor" value="50"></property>
 
        //另外一种RingBuffer填充时机, 在Schedule线程中, 周期性检查填充
        //默认:不配置此项, 即不实用Schedule线程. 如需使用, 请指定Schedule线程时间间隔, 单位:秒
        cachedUidGenerator.setScheduleInterval(60L);
 
 
 
        //拒绝策略: 当环已满, 无法继续填充时
        //默认无需指定, 将丢弃Put操作, 仅日志记录. 如有特殊需求, 请实现RejectedPutBufferHandler接口(支持Lambda表达式)
        //<property name="rejectedPutBufferHandler" ref="XxxxYourPutRejectPolicy"></property>
        //cachedUidGenerator.setRejectedPutBufferHandler();
        //拒绝策略: 当环已空, 无法继续获取时 -->
        //默认无需指定, 将记录日志, 并抛出UidGenerateException异常. 如有特殊需求, 请实现RejectedTakeBufferHandler接口(支持Lambda表达式) -->
        //<property name="rejectedTakeBufferHandler" ref="XxxxYourTakeRejectPolicy"></property>
 
        return cachedUidGenerator;
    }
 
}
```



## 四.在`renren-api`的`io.renren.dao`的文件中增加文件

WorkerNodeDao.java

```java
/*
 * Copyright (c) 2017 Baidu, Inc. All Rights Reserve.
 *
 * Licensed under the Apache License, Version 2.0 (the "License");
 * you may not use this file except in compliance with the License.
 * You may obtain a copy of the License at
 *
 *     http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */
package io.renren.dao;

import org.apache.ibatis.annotations.Mapper;
import org.apache.ibatis.annotations.Param;

import com.baidu.fsg.uid.worker.entity.WorkerNodeEntity;

/**
 * DAO for M_WORKER_NODE
 *
 * @author yutianbao
 */
@Mapper
public interface WorkerNodeDao {

    /**
     * Get {@link WorkerNodeEntity} by node host
     * 
     * @param host
     * @param port
     * @return
     */
    WorkerNodeEntity getWorkerNodeByHostPort(@Param("host") String host, @Param("port") String port);

    /**
     * Add {@link WorkerNodeEntity}
     * 
     * @param workerNodeEntity
     */
    void addWorkerNode(WorkerNodeEntity workerNodeEntity);

}
```



## 五.在`renren-api`的`io.renren.service`的文件中增加文件

```java
/*
 * Copyright (c) 2017 Baidu, Inc. All Rights Reserve.
 *
 * Licensed under the Apache License, Version 2.0 (the "License");
 * you may not use this file except in compliance with the License.
 * You may obtain a copy of the License at
 *
 *     http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */
package io.renren.service;

import javax.annotation.Resource;

import org.apache.commons.lang.math.RandomUtils;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.transaction.annotation.Transactional;

import com.baidu.fsg.uid.utils.DockerUtils;
import com.baidu.fsg.uid.utils.NetUtils;
import com.baidu.fsg.uid.worker.WorkerIdAssigner;
import com.baidu.fsg.uid.worker.WorkerNodeType;
import com.baidu.fsg.uid.worker.entity.WorkerNodeEntity;

import io.renren.dao.WorkerNodeDao;

/**
 * Represents an implementation of {@link WorkerIdAssigner}, 
 * the worker id will be discarded after assigned to the UidGenerator
 * 
 * @author yutianbao
 */
public class DisposableWorkerIdAssigner implements WorkerIdAssigner {
    private static final Logger LOGGER = LoggerFactory.getLogger(DisposableWorkerIdAssigner.class);

    @Resource
    private WorkerNodeDao workerNodeDao;

    /**
     * Assign worker id base on database.<p>
     * If there is host name & port in the environment, we considered that the node runs in Docker container<br>
     * Otherwise, the node runs on an actual machine.
     * 
     * @return assigned worker id
     */
    @Transactional
    public long assignWorkerId() {
        // build worker node entity
        WorkerNodeEntity workerNodeEntity = buildWorkerNode();

        // add worker node for new (ignore the same IP + PORT)
        workerNodeDao.addWorkerNode(workerNodeEntity);
        LOGGER.info("Add worker node:" + workerNodeEntity);

        return workerNodeEntity.getId();
    }

    /**
     * Build worker node entity by IP and PORT
     */
    private WorkerNodeEntity buildWorkerNode() {
        WorkerNodeEntity workerNodeEntity = new WorkerNodeEntity();
        if (DockerUtils.isDocker()) {
            workerNodeEntity.setType(WorkerNodeType.CONTAINER.value());
            workerNodeEntity.setHostName(DockerUtils.getDockerHost());
            workerNodeEntity.setPort(DockerUtils.getDockerPort());

        } else {
            workerNodeEntity.setType(WorkerNodeType.ACTUAL.value());
            workerNodeEntity.setHostName(NetUtils.getLocalAddress());
            workerNodeEntity.setPort(System.currentTimeMillis() + "-" + RandomUtils.nextInt(100000));
        }

        return workerNodeEntity;
    }

}
```



## 六.在`renren-api`的`src/main/resources/mapper/`的文件中增加文件`WorkerNodeDao.xml`

WorkerNodeDao.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="io.renren.dao.WorkerNodeDao">
	<resultMap id="workerNodeRes"
			   type="com.baidu.fsg.uid.worker.entity.WorkerNodeEntity">
		<id column="ID" jdbcType="BIGINT" property="id" />
		<result column="HOST_NAME" jdbcType="VARCHAR" property="hostName" />
		<result column="PORT" jdbcType="VARCHAR" property="port" />
		<result column="TYPE" jdbcType="INTEGER" property="type" />
		<result column="LAUNCH_DATE" jdbcType="DATE" property="launchDate" />
		<result column="MODIFIED" jdbcType="TIMESTAMP" property="modified" />
		<result column="CREATED" jdbcType="TIMESTAMP" property="created" />
	</resultMap>

	<insert id="addWorkerNode" useGeneratedKeys="true" keyProperty="id"
		parameterType="com.baidu.fsg.uid.worker.entity.WorkerNodeEntity">
		INSERT INTO WORKER_NODE
		(HOST_NAME,
		PORT,
		TYPE,
		LAUNCH_DATE,
		MODIFIED,
		CREATED)
		VALUES (
		#{hostName},
		#{port},
		#{type},
		#{launchDate},
		NOW(),
		NOW())
	</insert>

	<select id="getWorkerNodeByHostPort" resultMap="workerNodeRes">
		SELECT
		ID,
		HOST_NAME,
		PORT,
		TYPE,
		LAUNCH_DATE,
		MODIFIED,
		CREATED
		FROM
		WORKER_NODE
		WHERE
		HOST_NAME = #{host} AND PORT = #{port}
	</select>
</mapper>
```

## 七.在`renren-api`的Controller中编写生成方法

ApiTestController.java

```java
/**
 * Copyright (c) 2016-2019 人人开源 All rights reserved.
 *
 * https://www.renren.io
 *
 * 版权所有，侵权必究！
 */

package io.renren.controller;

import io.renren.annotation.Login;
import io.renren.annotation.LoginUser;
import io.renren.common.utils.R;
import io.renren.entity.UserEntity;
import io.swagger.annotations.Api;
import io.swagger.annotations.ApiOperation;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestAttribute;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import com.baidu.fsg.uid.UidGenerator;

import springfox.documentation.annotations.ApiIgnore;

/**
 * 测试接口
 *
 * @author Mark sunlightcs@gmail.com
 */
@RestController
@RequestMapping("/api")
@Api(tags="测试接口")
public class ApiTestController {

    @Autowired
    private UidGenerator uidGenerator;
    
    @Login
    @GetMapping("userInfo")
    @ApiOperation(value="获取用户信息", response=UserEntity.class)
    public R userInfo(@ApiIgnore @LoginUser UserEntity user){
        return R.ok().put("user", user);
    }

    @Login
    @GetMapping("userId")
    @ApiOperation("获取用户ID")
    public R userInfo(@ApiIgnore @RequestAttribute("userId") Integer userId){
        return R.ok().put("userId", userId);
    }

    @GetMapping("notToken")
    @ApiOperation("忽略Token验证测试")
    public R notToken(){
        return R.ok().put("msg", "无需token也能访问。。。");
    }
    
    @GetMapping("uuid")
    @ApiOperation("百度生成uid")
    public R uidGenerator(){
        return R.ok().put("uid", Long.toString(uidGenerator.getUID()));
    }
}
```



## 七.在`renren-api`中执行`ApiApplication.java`

项目运行后访问地址：<http://localhost:8081/renren-api/api/uuid>

访问结果：

```json
{"msg":"success","uid":"1340011929288130560","code":0}
```



=======================================================================================

博客地址：<https://www.codepeople.cn>

=======================================================================================

微信公众号：

![](https://www.codepeople.cn/imges/weixin_icon/weixin.jpg)