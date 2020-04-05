---
title:  "Alibaba 开源通用缓存访问框架：JetCache"
categories: JetCache
tags: JetCache
author: LIUREN
---

# Alibaba 开源通用缓存访问框架：JetCache

> JetCache
> 是由阿里巴巴开源的一款通用缓存访问框架。
> 我们来了解下这款更好用的 JetCache。
> 作者：liuren
>
> 链接：https://www.codepeople.cn
>
> 来源：GitHub
>
> 著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。



引用下官方文档说明，JetCache 提供的核心能力包括：

- 提供统一的，类似jsr-107风格的API访问Cache，并可通过注解创建并配置Cache实例
- 通过注解实现声明式的方法缓存，支持TTL和两级缓存
- 分布式缓存自动刷新，分布式锁 (2.2+)
- 支持异步Cache API
- Spring Boot支持
- Key的生成策略和Value的序列化策略是可以定制的
- 针对所有Cache实例和方法缓存的自动统计

目前支持的缓存系统包括以下4个：

- Caffeine（基于本地内存）
- LinkedHashMap（基于本地内存，JetCache自己实现的简易LRU缓存）
- Alibaba Tair（相关实现未在Github开源，在阿里内部Gitlab上可以找到）
- Redis

来看个简单的使用示例：

```java
public interface UserService {

    @Cached(name="user", key="#userId", expire = 3600, cacheType = CacheType.REMOTE)
	@CacheRefresh(refresh = 1800, stopRefreshAfterLastAccess = 3600, timeUnit = TimeUnit.SECONDS)
	@CachePenetrationProtect
    User getById(long userId);
	
}
```

是不是和 Spring Cache 很像，不过这里原生支持了细粒度的 TTL（超时时间，而 Spring Cache Redis 默认只支持配置全局的），CacheType 还有 LOCAL/REMOTE/BOTH 三种选择， 分别代表本地内存/远程 Cache Server（如Redis）/两级缓存。下面，结合 Redis 的使用，来看看更复杂的一些使用场景：

### Example

首先，使用 JetCache 的环境需求包括如下：

- JDK：必须 Java 8+
- Spring Framework：v4.0.8 以上，如果不使用注解就不需要
- Spring Boot：v1.1.9 以上（可选）

1. 依赖部分，示例中使用的是基于 Lettuce Redis 客户端的 Spring Boot 方式的 Starter

```xml
<dependencies>
    <!-- https://mvnrepository.com/artifact/com.alicp.jetcache/jetcache-redis -->
    <dependency>
        <groupId>com.alicp.jetcache</groupId>
        <artifactId>jetcache-starter-redis-lettuce</artifactId>
        <version>2.6.0.M1</version>
    </dependency>

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>

    <dependency>
        <groupId>com.h2database</groupId>
        <artifactId>h2</artifactId>
        <scope>runtime</scope>
    </dependency>
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <optional>true</optional>
    </dependency>
</dependencies>
```

**插件这部分的配置是必需的，参考下方引用：**

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-compiler-plugin</artifactId>
            <version>3.7.0</version>
            <configuration>
                <source>1.8</source>
                <target>1.8</target>
                <compilerArgument>-parameters</compilerArgument>
            </configuration>
        </plugin>
    </plugins>
</build>
```

[@Cached的key、condition等表达式中使用参数名以后缓存没有生效？](https://github.com/alibaba/jetcache/wiki/FAQ_CN#cached%E7%9A%84keycondition%E7%AD%89%E8%A1%A8%E8%BE%BE%E5%BC%8F%E4%B8%AD%E4%BD%BF%E7%94%A8%E5%8F%82%E6%95%B0%E5%90%8D%E4%BB%A5%E5%90%8E%E7%BC%93%E5%AD%98%E6%B2%A1%E6%9C%89%E7%94%9F%E6%95%88)

1. 全局配置

```yaml
spring:
  jpa:
    database-platform: org.hibernate.dialect.H2Dialect
    hibernate:
      ddl-auto: create
    # 开启 SQL 输出，方便查看结果是否走了缓存
    show-sql: true

# @see com.alicp.jetcache.autoconfigure.JetCacheProperties
jetcache:
  # 统计间隔，默认0：表示不统计
  statIntervalMinutes: 1
  # areaName是否作为缓存key前缀，默认True
  areaInCacheName: false
  local:
    default:
      # 已支持可选：linkedhashmap、caffeine
      type: linkedhashmap
      # key转换器的全局配置，当前只有：fastjson, @see com.alicp.jetcache.support.FastjsonKeyConvertor
      keyConvertor: fastjson
      # 每个缓存实例的最大元素的全局配置，仅local类型的缓存需要指定
      limit: 100
      # jetcache2.2以上，以毫秒为单位，指定多长时间没有访问，就让缓存失效，当前只有本地缓存支持。0表示不使用这个功能
      expireAfterAccessInMillis: 30000
  remote:
    default:
      # 已支持可选：redis、tair
      type: redis.lettuce
      # 连接格式@see：https://github.com/lettuce-io/lettuce-core/wiki/Redis-URI-and-connection-details
      uri: redis://localhost:6379/1?timeout=5s
      keyConvertor: fastjson
      # 序列化器的全局配置。仅remote类型的缓存需要指定，可选java和kryo
      valueEncoder: java
      valueDecoder: java
      # 以毫秒为单位指定超时时间的全局配置
      expireAfterWriteInMillis: 5000
```

1. 编写实体

```java
@Data
@Builder
@NoArgsConstructor
@AllArgsConstructor
@Entity
public class Coffee implements Serializable {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Integer id;

    private String name;

    private Float price;

}
```

```java
public interface CoffeeRepository extends JpaRepository<Coffee, Integer> {
}
```

JetCache 提供了两种实现缓存的方式，可以通过 `@CreateCache` 注解创建 Cache 实例，这种灵活性比较高；其次是通过纯注解的方式来实现缓存。

4.1 通过 @CreateCache 注解创建 Cache 实例

```java
@Slf4j
@Service
public class CoffeeCreateCacheService {

    private static final String CACHE_NAME = "CoffeeCreateCache:";

    @Resource
    private CoffeeRepository coffeeRepository;

     /**
     * 使用 @CreateCache 注解创建Cache实例;
     * 未定义默认值的参数，将使用yml中指定的全局配置;
     * 缓存在 Local，也可以配置成 both 开启两级缓存
     */
    @CreateCache(name = CACHE_NAME, expire = 1, localLimit = 10,
            timeUnit = TimeUnit.MINUTES, cacheType = CacheType.LOCAL)
    private Cache<Integer, Coffee> coffeeCache;

    @Transactional
    public void add(Coffee coffee) {
        coffeeRepository.save(coffee);
        coffeeCache.put(coffee.getId(), coffee, 3, TimeUnit.SECONDS);
    }

    public Optional<Coffee> get(int id) {
        Coffee coffee = coffeeCache.get(id);
        log.info("CoffeeCreateCache get {} res {}", id, coffee);
        if (Objects.isNull(coffee)) {
            Optional<Coffee> coffeeOptional = coffeeRepository.findById(id);
            if (coffeeOptional.isPresent()) {
                coffee = coffeeOptional.get();
                boolean res = coffeeCache.putIfAbsent(id, coffee);
                log.info("CoffeeCreateCache putIfAbsent {} res {}", id, res);
            }
        }
        return Optional.ofNullable(coffee);
    }

    @Transactional
    public Coffee update(Coffee coffee) {
        Coffee c = coffeeRepository.save(coffee);
        coffeeCache.put(c.getId(), c, 60, TimeUnit.SECONDS);
        return c;
    }

    @Transactional
    public void delete(int id) {
        coffeeRepository.deleteById(id);
        boolean res = coffeeCache.remove(id);
        log.info("CoffeeCreateCache delete {} res {}", id, res);
    }

}
```

4.2 通过注解实现方法缓存，主要是：`@Cached`(缓存新增)、`@CacheUpdate`(缓存更新)、`@CacheInvalidate`(缓存删除)，还有用于配置自动刷新和加载保护的“大杀器”：`@CacheRefresh`、`@CachePenetrationProtect`

```java
@Slf4j
@Service
public class CoffeeMethodCacheService {

    private static final String CACHE_NAME = "CoffeeMethodCache:";

    @Resource
    private CoffeeRepository coffeeRepository;

    @Transactional
    public Coffee add(Coffee coffee) {
        return coffeeRepository.save(coffee);
    }
	
	/**
     * 缓存在 Remote 的 Redis，也可以配置成 both 开启两级缓存
     */
    @Cached(name = CACHE_NAME, key = "#id", cacheType = CacheType.REMOTE, serialPolicy = SerialPolicy.KRYO,
            condition = "#id>0", postCondition = "result!=null")
    public Coffee get(int id) {
        return coffeeRepository.findById(id).orElse(null);
    }

    @CacheUpdate(name = CACHE_NAME, key = "#coffee.id", value = "result", condition = "#coffee.id!=null")
    @Transactional
    public Coffee update(Coffee coffee) {
        return coffeeRepository.save(coffee);
    }

    @CacheInvalidate(name = CACHE_NAME, key = "#id")
    @Transactional
    public void delete(int id) {
        coffeeRepository.deleteById(id);
    }

}
```

1. 简单测试

```java
/**
 * 这里 @EnableMethodCache，@EnableCreateCacheAnnotation 分别用于激活 @Cached 和 @CreateCache 注解
 */
@Slf4j
@EnableMethodCache(basePackages = "cn.mariojd.jetcache")
@EnableCreateCacheAnnotation
@SpringBootApplication
public class SpirngBootJetcacheApplication implements ApplicationRunner {

    public static void main(String[] args) {
        new SpringApplicationBuilder()
                .sources(SpirngBootJetcacheApplication.class)
                .bannerMode(Banner.Mode.OFF)
                .web(WebApplicationType.NONE)
                .run(args);
    }

    @Resource
    private CoffeeCreateCacheService coffeeCreateCacheService;

    @Resource
    private CoffeeMethodCacheService coffeeMethodCacheService;

    @Override
    public void run(ApplicationArguments args) throws InterruptedException {
        // Test Coffee create cache

        Coffee latte = Coffee.builder().name("Latte").price(20.0f).build();
        coffeeCreateCacheService.add(latte);
        log.info("Reading from cache... {}", coffeeCreateCacheService.get(latte.getId()));
        TimeUnit.SECONDS.sleep(3);
        log.info("Cache expire... ");
        coffeeCreateCacheService.get(latte.getId());
        latte.setPrice(25.0f);
        latte = coffeeCreateCacheService.update(latte);
        coffeeCreateCacheService.delete(latte.getId());

        // Test Coffee method cache

        Coffee cappuccino = Coffee.builder().name("Cappuccino").price(30.0f).build();
        coffeeMethodCacheService.add(cappuccino);
        coffeeMethodCacheService.get(cappuccino.getId());
        log.info("Reading from cache... {}", coffeeMethodCacheService.get(cappuccino.getId()));
        cappuccino.setPrice(25.0f);
        cappuccino = coffeeMethodCacheService.update(cappuccino);
        coffeeMethodCacheService.delete(cappuccino.getId());
    }
    
}
```

具体的可以自行运行测试，最后来看看 Redis 的 Monitor，图中红框部分说明该查询走了缓存：

![](https://www.codepeople.cn/imges/20190410141540.png)

原文地址：<https://blog.mariojd.cn/spring-boot-jetcache-example.html>

博客地址：<https://www.codepeople.cn>

=====================================================================

微信公众号：

![](https://www.codepeople.cn/imges/weixin_icon/weixin.jpg)