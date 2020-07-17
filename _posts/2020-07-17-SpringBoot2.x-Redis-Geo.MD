---
title:  "使用SpringBoot和Redis的Geo功能获取当前位置方圆一公里的坐标点"
categories: SpringBoot Redis Geo
tags: SpringBoot Redis Geo
author: LIUREN
---

# 使用SpringBoot和Redis的Geo功能获取当前位置方圆一公里的坐标点

> 使用SpringBoot和Redis的Geo功能获取当前位置方圆一公里的坐标点。使用场景是这样，如果通过手机坐标获取当前位置，然后当前位置1公里内的所有的参数的坐标会返回给前端并在地图展示
>
> 协议：CC BY-SA 4.0 https://creativecommons.org/licenses/by-sa/4.0/  
>
> 版权声明：本文为原创文章，遵循 CC 4.0 BY-SA 版权协议，转载请附上原文出处链接及本声明。



## 第一步：Redis 中的GEO

**Redis**是我们最为熟悉的**K-V**数据库，它常被拿来作为高性能的缓存数据库来使用，大部分项目都会用到它。从**3.2**版本开始它开始提供了**GEO**能力，用来实现诸如附近位置、计算距离等这类依赖于地理位置信息的功能。**GEO**相关的命令如下：

| Redis命令         | 描述                                                      |
| :---------------- | :-------------------------------------------------------- |
| GEOHASH           | 返回一个或多个位置元素的 **Geohash** 表示                 |
| GEOPOS            | 从key里返回所有给定位置元素的位置（经度和纬度）           |
| GEODIST           | 返回两个给定位置之间的距离                                |
| GEORADIUS         | 以给定的经纬度为中心， 找出某一半径内的元素               |
| GEOADD            | 将指定的地理空间位置（纬度、经度、名称）添加到指定的key中 |
| GEORADIUSBYMEMBER | 找出位于指定范围内的元素，中心点是由给定的位置元素决定    |

Redis会假设地球为完美的球形, 所以可能有一些位置计算偏差，据说<=0.5%，对于有严格地理位置要求的需求来说要经过一些场景测试来检验是否能够满足需求。

## 第二步：写入地理信息

那么如何实现目标单位半径内的所有元素呢？我们可以将所有的位置的经纬度通过上表中的`GEOADD`将这些地理信息转换为52位的**Geohash**写入**Redis**。

格式：

```bash
geoadd key longitude latitude member [longitude latitude member ...]
```

例子：

```bash
redis> geoadd cities:locs 117.12 39.08 tianjin 114.29 38.02  shijiazhuang 
(integer) 2
```

意思是将经度为`117.12`纬度为`39.08`的地点`tianjin`和经度为`114.29`纬度为`38.02`的地点`shijiazhuang`加入**key**为`cities:locs`的 **sorted set**集合中。可以添加一到多个位置。然后我们就可以借助于其他命令来进行地理位置的计算了。

> 有效的经度从-180度到180度。有效的纬度从-85.05112878度到85.05112878度。当坐标位置超出上述指定范围时，该命令将会返回一个错误。

###  统计单位半径内的地区

我们可以借助于`GEORADIUS`来找出以给定经纬度，某一半径内的所有元素。

该命令格式

```bash
georadius key longtitude latitude radius m|km|ft|mi [WITHCOORD] [WITHDIST] [WITHHASH] [COUNT count] [ASC|DESC] 
```

这个命令比`GEOADD`要复杂一些：

- **radius**  半径长度，必选项。后面的`m`、`km`、`ft`、`mi`、是长度单位选项，四选一。
- **WITHCOORD**  将位置元素的经度和维度也一并返回，非必选。
- **WITHDIST** 在返回位置元素的同时， 将位置元素与中心点的距离也一并返回。 距离的单位和查询单位一致，非必选。
- **WITHHASH** 返回位置的52位精度的**Geohash**值，非必选。这个我反正很少用，可能其它一些偏向底层的**LBS**应用服务需要这个。
- **COUNT** 返回符合条件的位置元素的数量，非必选。比如返回前10个，以避免出现符合的结果太多而出现性能问题。
- **ASC|DESC** 排序方式，非必选。默认情况下返回未排序，但是大多数我们需要进行排序。参照中心位置，从近到远使用**ASC** ，从远到近使用**DESC**。

例如，我们在 `cities:locs` 中查找以（115.03，38.44）为中心，方圆`200km`的城市，结果包含城市名称、对应的坐标和距离中心点的距离（km），并按照从近到远排列。命令如下：

```bash
redis> georadius cities:locs 115.03 38.44 200 km WITHCOORD WITHDIST ASC
1) 1) "shijiazhuang"
   2) "79.7653"
   3) 1) "114.29000169038772583"
      2) "38.01999994251037407"
2) 1) "tianjin"
   2) "186.6937"
   3) 1) "117.02000230550765991"
      2) "39.0800000535766543"
```

你可以加上 `COUNT 1`来查找最近的一个位置。

## 第三步：进入开发阶段

![](https://www.codepeople.cn/imges/00x37.png)

![](https://www.codepeople.cn/imges/00x38.png)



开发环境

| 开发平台  | Windows 10          |
| --------- | ------------------- |
| 开发语言  | Java                |
| 开发框架  | SpringBoot2 、Redis |
| Redis版本 | 5.0.9               |

1.pom.xml代码

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.example</groupId>
    <artifactId>redis-geo</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>redis-geo</name>
    <description>使用Redis的GEO获取两地范围内的信息</description>

    <properties>
        <java.version>1.8</java.version>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <spring-boot.version>2.3.0.RELEASE</spring-boot.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
        </dependency>
		<dependency>
		        <groupId>org.springframework.boot</groupId>
		        <artifactId>spring-boot-starter-data-redis</artifactId>
		</dependency>
		<!-- https://mvnrepository.com/artifact/org.projectlombok/lombok -->
		<dependency>
		    <groupId>org.projectlombok</groupId>
		    <artifactId>lombok</artifactId>
		    <scope>provided</scope>
		</dependency>
		<!-- https://mvnrepository.com/artifact/com.alibaba/fastjson -->
		<dependency>
		    <groupId>com.alibaba</groupId>
		    <artifactId>fastjson</artifactId>
		    <version>1.2.72</version>
		</dependency>
		<!-- https://mvnrepository.com/artifact/junit/junit -->
		<dependency>
		    <groupId>junit</groupId>
		    <artifactId>junit</artifactId>
		    <scope>test</scope>
		</dependency>
		
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
            <exclusions>
                <exclusion>
                    <groupId>org.junit.vintage</groupId>
                    <artifactId>junit-vintage-engine</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
    </dependencies>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-dependencies</artifactId>
                <version>${spring-boot.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <configuration>
                    <source>1.8</source>
                    <target>1.8</target>
                    <encoding>UTF-8</encoding>
                </configuration>
            </plugin>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>
```

2.实体类CityInfo.java

```java
package com.example.redisgeo.bean;

import java.io.Serializable;

import lombok.AllArgsConstructor;
import lombok.Builder;
import lombok.Data;
import lombok.NoArgsConstructor;

@Data
@Builder
@NoArgsConstructor
@AllArgsConstructor
public class CityInfo implements Serializable {

	
	private static final long serialVersionUID = 1L;

	/** 城市 */
    private String city;

    /** 经度 */
    private Double longitude;

    /** 纬度 */
    private Double latitude;
    
}
```

3.服务接口IGeoService.java

```java
package com.example.redisgeo.service;

import java.util.Collection;
import java.util.List;

import org.springframework.data.geo.Circle;
import org.springframework.data.geo.Distance;
import org.springframework.data.geo.GeoResults;
import org.springframework.data.geo.Metric;
import org.springframework.data.geo.Point;
import org.springframework.data.redis.connection.RedisGeoCommands;

import com.example.redisgeo.bean.CityInfo;

public interface IGeoService {
	
	/**
	 * @Title: saveCityInfoToRedis
	 * @Description:  把城市信息保存到 Redis 中
	 * @Author 刘 仁
	 * @DateTime 2020年7月17日 下午3:45:30
	 * @param cityInfos
	 * @return 成功保存的个数
	 */
	Long saveCityInfoToRedis(Collection<CityInfo> cityInfos);
	
	/**
	 * @Title: getCityPos
	 * @Description: 获取给定城市的坐标
	 * @Author 刘 仁
	 * @DateTime 2020年7月17日 下午3:46:22
	 * @param  cities 给定城市 key
	 * @return
	 */
	List<Point> getCityPos(String[] cities);
	
	/**
	 * @Title: getTwoCityDistance
	 * @Description: 获取两个城市之间的距离
	 * @Author 刘 仁
	 * @DateTime 2020年7月17日 下午3:46:47
	 * @param city1 第一个城市
	 * @param city2 第二个城市
	 * @param metric 单位信息, 可以是 null
	 * @return
	 */
	Distance getTwoCityDistance(String city1, String city2, Metric metric);
	
	/**
	 * @Title: getPointRadius
	 * @Description: 根据给定地理位置坐标获取指定范围内的地理位置集合
	 * @Author 刘 仁
	 * @DateTime 2020年7月17日 下午3:47:38
	 * @param within 中心点和距离
	 * @param args 限制返回的个数和排序方式, 可以是 null
	 * @return
	 */
	GeoResults<RedisGeoCommands.GeoLocation<String>> getPointRadius(
            Circle within, RedisGeoCommands.GeoRadiusCommandArgs args);
	
	/**
	 * @Title: getMemberRadius
	 * @Description: 根据给定地理位置获取指定范围内的地理位置集合
	 * @Author 刘 仁
	 * @DateTime 2020年7月17日 下午3:48:12
	 * @param member
	 * @param distance
	 * @param args
	 * @return
	 */
	GeoResults<RedisGeoCommands.GeoLocation<String>> getMemberRadius(
            String member, Distance distance, RedisGeoCommands.GeoRadiusCommandArgs args);

	/**
	 * @Title: getCityGeoHash
	 * @Description: 获取某个地理位置的 geohash 值
	 * @Author 刘 仁
	 * @DateTime 2020年7月17日 下午3:48:29
	 * @param cities 给定城市 key
	 * @return
	 */
	List<String> getCityGeoHash(String[] cities);
}
```

4.服务实现类GeoServiceImpl.java

```java
package com.example.redisgeo.service.impl;


import java.util.Collection;
import java.util.HashSet;
import java.util.List;
import java.util.Set;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.geo.Circle;
import org.springframework.data.geo.Distance;
import org.springframework.data.geo.GeoResults;
import org.springframework.data.geo.Metric;
import org.springframework.data.geo.Point;
import org.springframework.data.redis.connection.RedisGeoCommands;
import org.springframework.data.redis.connection.RedisGeoCommands.GeoLocation;
import org.springframework.data.redis.connection.RedisGeoCommands.GeoRadiusCommandArgs;
import org.springframework.data.redis.core.GeoOperations;
import org.springframework.data.redis.core.StringRedisTemplate;
import org.springframework.stereotype.Service;

import com.alibaba.fastjson.JSON;
import com.example.redisgeo.bean.CityInfo;
import com.example.redisgeo.service.IGeoService;

import lombok.extern.slf4j.Slf4j;

@Slf4j
@Service
public class GeoServiceImpl implements IGeoService{

	private final String GEO_KEY = "ah-cities";
	
	/** redis 客户端 */
    private final StringRedisTemplate redisTemplate;
    
    @Autowired
    public GeoServiceImpl(StringRedisTemplate redisTemplate) {
        this.redisTemplate = redisTemplate;
    }
    
	@Override
	public Long saveCityInfoToRedis(Collection<CityInfo> cityInfos) {
		log.info("start to save city info: {}.", JSON.toJSONString(cityInfos));
        GeoOperations<String, String> ops = redisTemplate.opsForGeo();
        Set<RedisGeoCommands.GeoLocation<String>> locations = new HashSet<>();
        cityInfos.forEach(ci -> locations.add(new RedisGeoCommands.GeoLocation<String>(
                ci.getCity(), new Point(ci.getLongitude(), ci.getLatitude())
        )));
        log.info("done to save city info.");
        return ops.add(GEO_KEY, locations);
	}

	@Override
	public List<Point> getCityPos(String[] cities) {
		GeoOperations<String, String> ops = redisTemplate.opsForGeo();
        return ops.position(GEO_KEY, cities);
	}

	@Override
	public Distance getTwoCityDistance(String city1, String city2, Metric metric) {
		GeoOperations<String, String> ops = redisTemplate.opsForGeo();
        return metric == null ? ops.distance(GEO_KEY, city1, city2) : ops.distance(GEO_KEY, city1, city2, metric);
	}

	@Override
	public GeoResults<GeoLocation<String>> getPointRadius(Circle within, GeoRadiusCommandArgs args) {
		GeoOperations<String, String> ops = redisTemplate.opsForGeo();
        return args == null ? ops.radius(GEO_KEY, within) : ops.radius(GEO_KEY, within, args);
	}

	@Override
	public GeoResults<GeoLocation<String>> getMemberRadius(String member, Distance distance,
			GeoRadiusCommandArgs args) {
		GeoOperations<String, String> ops = redisTemplate.opsForGeo();
        return args == null ? ops.radius(GEO_KEY, member, distance) : ops.radius(GEO_KEY, member, distance, args);
	}

	@Override
	public List<String> getCityGeoHash(String[] cities) {
		GeoOperations<String, String> ops = redisTemplate.opsForGeo();
        return ops.hash(GEO_KEY, cities);
	}

}
```

5.配置文件

```properties
spring.application.name=redis-geo
# Redis数据库索引（默认为0）
spring.redis.database=0
# Redis服务器地址
spring.redis.host=127.0.0.1
# Redis服务器连接端口
spring.redis.port=6379
# Redis服务器连接密码（默认为空）
spring.redis.password=
# 连接池最大连接数（使用负值表示没有限制）
spring.redis.jedis.pool.max-active=20
# 连接池最大阻塞等待时间（使用负值表示没有限制）
spring.redis.jedis.pool.max-wait=-1
# 连接池中的最大空闲连接
spring.redis.jedis.pool.max-idle=10
# 连接池中的最小空闲连接
spring.redis.jedis.pool.min-idle=0
# 连接超时时间（毫秒）
spring.redis.timeout=1000
```

6.单元测试类

```java
package com.example.redisgeo;

import com.alibaba.fastjson.JSON;
import com.example.redisgeo.bean.CityInfo;
import com.example.redisgeo.service.IGeoService;

import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.data.geo.Circle;
import org.springframework.data.geo.Distance;
import org.springframework.data.geo.Metrics;
import org.springframework.data.geo.Point;
import org.springframework.data.redis.connection.RedisGeoCommands;
import org.springframework.test.context.junit4.SpringRunner;

import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;

/**
 * @ClassName: RedisGeoApplicationTests
 * @Description: 测试用例
 * @Author 刘 仁
 * @DateTime 2020年7月17日 下午3:59:24
 */
@RunWith(SpringRunner.class)
@SpringBootTest(classes = {RedisGeoApplication.class}, webEnvironment = SpringBootTest.WebEnvironment.NONE)
public class RedisGeoApplicationTests {

    /** fake some cityInfos */
    private List<CityInfo> cityInfos;

    @Autowired
    private IGeoService geoService;

    @BeforeEach
    public void init() {

        cityInfos = new ArrayList<>();

        cityInfos.add(new CityInfo("hefei", 117.17, 31.52));
        cityInfos.add(new CityInfo("anqing", 117.02, 30.31));
        cityInfos.add(new CityInfo("huaibei", 116.47, 33.57));
        cityInfos.add(new CityInfo("suzhou", 116.58, 33.38));
        cityInfos.add(new CityInfo("fuyang", 115.48, 32.54));
        cityInfos.add(new CityInfo("bengbu", 117.21, 32.56));
        cityInfos.add(new CityInfo("huangshan", 118.18, 29.43));
    }

   /**
    * @Title: testSaveCityInfoToRedis
    * @Description: 测试把所有的标准的坐标信息放到redis中
    * @Author 刘 仁
    * @DateTime 2020年7月17日 下午4:46:40
    */
    @Test
    public void testSaveCityInfoToRedis() {

        System.out.println(geoService.saveCityInfoToRedis(cityInfos));
    }

    /**
     * @Title: testGetCityPos
     * @Description: 获取给定城市的坐标
     * @Author 刘 仁
     * @DateTime 2020年7月17日 下午4:47:41
     */
    @Test
    public void testGetCityPos() {

        System.out.println(JSON.toJSONString(geoService.getCityPos(
                Arrays.asList("anqing", "suzhou", "xxx").toArray(new String[3])
        )));
    }

   /**
    * @Title: testGetTwoCityDistance
    * @Description: 获取两个城市之间的距离
    * @Author 刘 仁
    * @DateTime 2020年7月17日 下午4:48:03
    */
    @Test
    public void testGetTwoCityDistance() {

        System.out.println(geoService.getTwoCityDistance("anqing", "suzhou", null).getValue());
        System.out.println(geoService.getTwoCityDistance("anqing", "suzhou", Metrics.KILOMETERS).getValue());
    }

    /**
     * @Title: testGetPointRadius
     * @Description: 根据给定地理位置坐标获取指定范围内的地理位置集合
     * @Author 刘 仁
     * @DateTime 2020年7月17日 下午4:48:28
     */
    @Test
    public void testGetPointRadius() {

        Point center = new Point(cityInfos.get(0).getLongitude(), cityInfos.get(0).getLatitude());
        Distance radius = new Distance(200, Metrics.KILOMETERS);
        Circle within = new Circle(center, radius);

        System.out.println(JSON.toJSONString(geoService.getPointRadius(within, null)));

        // order by 距离 limit 2, 同时返回距离中心点的距离
        RedisGeoCommands.GeoRadiusCommandArgs args =
                RedisGeoCommands.GeoRadiusCommandArgs.newGeoRadiusArgs().includeDistance().limit(2).sortAscending();
        System.out.println(JSON.toJSONString(geoService.getPointRadius(within, args)));
    }

    /**
     * @Title: testGetMemberRadius
     * @Description: 根据给定地理位置获取指定范围内的地理位置集合
     * @Author 刘 仁
     * @DateTime 2020年7月17日 下午4:49:24
     */
    @Test
    public void testGetMemberRadius() {

        Distance radius = new Distance(200, Metrics.KILOMETERS);

        System.out.println(JSON.toJSONString(geoService.getMemberRadius("suzhou", radius, null)));

        // order by 距离 limit 2, 同时返回距离中心点的距离
        RedisGeoCommands.GeoRadiusCommandArgs args =
                RedisGeoCommands.GeoRadiusCommandArgs.newGeoRadiusArgs().includeDistance().limit(2).sortAscending();
        System.out.println(JSON.toJSONString(geoService.getMemberRadius("suzhou", radius, args)));
    }

    /**
     * @Title: testGetCityGeoHash
     * @Description: 获取某个地理位置的 geohash 值
     * @Author 刘 仁
     * @DateTime 2020年7月17日 下午4:50:05
     */
    @Test
    public void testGetCityGeoHash() {

        System.out.println(JSON.toJSONString(geoService.getCityGeoHash(
                Arrays.asList("anqing", "suzhou", "xxx").toArray(new String[3])
        )));
    }
}
```







博客地址：<https://www.codepeople.cn>

=====================================================================

微信公众号：

![](https://www.codepeople.cn/imges/weixin_icon/weixin.jpg)