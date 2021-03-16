---
title:  "通过静态内部类实现SnowFlake(雪花)算法"
categories: snowflake
tags: snowflake
author: LIUREN
---

# 通过静态内部类实现SnowFlake(雪花)算法

> 在生成表主键ID时，我们可以考虑主键自增或者UUID，但是他们都有明显的缺点；
>
> `主键自增`: 1.自增ID容易被爬虫遍历数据。2.分表分库会有ID冲突
>
> `UUID`： 1.太长，并且有索引碎片，索引多占用空间的问题 2.无序
>
> 雪花算法就很适合在分布式场景下生成唯一ID，**他可以保证唯一又可以排序**。为了提高生产雪花ID的效率，在这里数据的运算都采用的是位运算。
>
> 协议：CC BY-SA 4.0 https://creativecommons.org/licenses/by-sa/4.0/  
>
> 版权声明：本文为原创文章，遵循 CC 4.0 BY-SA 版权协议，转载请附上原文出处链接及本声明。



## 一、概念

### 1、原理

SnowFlake算法生成ID的结果是一个64bit大小的整数，它的结构如下图：

![](https://www.codepeople.cn/imges/00x102.png)

**算法描述：**

- **1bit** 因为二进制中最高位是符号位，1表示负数，0表示正数。生成的ID都是正整数，所以最高位固定为0。
- **41bit-时间戳** 精确到毫秒级，41位的长度可以使用69年。时间位还有一个很重要的作用是可以根据时间进行排序。
- **10bit-工作机器id** 10位的机器标识，10位的长度最多支持部署1024个节点。
- **12bit-序列号** 序列号即一系列的自增id，可以支持同一节点同一毫秒生成多个ID序号。
  12位（bit）可以表示的最大正整数是![2^{12}-1 = 4095](https://math.jianshu.com/math?formula=2%5E%7B12%7D-1%20%3D%204095)，即可以用0、1、2、3、....4094这4095个数字，来表示同一机器同一时间截（毫秒)内产生的4095个ID序号。

`说明` 由于在Java中64bit的整数是long类型,所以在Java中SnowFlake算法生成的id就是long来存储的。



## 二、静态类部类单例模式生产雪花ID代码

下面生成雪花ID的代码可以用语线上分布式项目中来生成主键ID，因为设计采用的静态内部类的单例模式，通过加`synchronized`锁来保证在同一个服务器线程安全。至于不同服务器其实是不相关的,因为它们的机器码是不一致的,所以就算同一时刻两台服务器都产生了雪花ID,那也不会一样的。

### 1.代码

SnowIdUtils.java

```java
package com.kelan.haiguan.util;

import java.util.Map;
import java.util.concurrent.ConcurrentHashMap;
import java.util.concurrent.CountDownLatch;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

/**
 * @ClassName: SnowIdUtils
 * @Description: 雪花算法(改造简化后的算法生成字符串为15位,原生算法是19位,主要区别是原生MACHINE_BIT被拆成了dataworkId(5位)+ workId(5位))
 * @DateTime 2021-3-15 16:30:17
 */
public class SnowIdUtils {
	
	private static final Logger log = LoggerFactory.getLogger(SnowIdUtils.class);
	/**
     * 私有的 静态内部类
     */
    private static class SnowFlake {

        /**
         * 内部类对象（单例模式）
         */
        private static final SnowIdUtils.SnowFlake SNOW_FLAKE = new SnowIdUtils.SnowFlake();
        /**
         * 起始的时间戳 2021-03-15 16:19:25
         */
        private final long START_TIMESTAMP = 1615796365175L;
        /**
         * 序列号占用位数
         */
        private final long SEQUENCE_BIT = 12;
        /**
         * 机器标识占用位数
         */
        private final long MACHINE_BIT = 10;
        /**
         * 时间戳位移位数
         */
        private final long TIMESTAMP_LEFT = SEQUENCE_BIT + MACHINE_BIT;
        /**
         * 最大序列号  （4095）
         */
        private final long MAX_SEQUENCE = ~(-1L << SEQUENCE_BIT);
        /**
         * 最大机器编号 （1023）
         */
        private final long MAX_MACHINE_ID = ~(-1L << MACHINE_BIT);
        /**
         * 生成id机器标识部分
         */
        private long machineIdPart;
        /**
         * 序列号
         */
        private long sequence = 0L;
        /**
         * 上一次时间戳
         */
        private long lastStamp = -1L;

        /**
         * 构造函数初始化机器编码
         */
        private SnowFlake() {
            //模拟这里获得本机机器编码
            long localIp = 4321;
            //localIp & MAX_MACHINE_ID最大不会超过1023,在左位移12位
            machineIdPart = (localIp & MAX_MACHINE_ID) << SEQUENCE_BIT;
        }
        /**
         * 获取雪花ID
         */
        public synchronized long nextId() {
            long currentStamp = timeGen();
            //避免机器时钟回拨
            while (currentStamp < lastStamp) {
                // //服务器时钟被调整了,ID生成器停止服务.
                throw new RuntimeException(String.format("时钟已经回拨.  Refusing to generate id for %d milliseconds", lastStamp - currentStamp));
            }
            if (currentStamp == lastStamp) {
                // 每次+1
                sequence = (sequence + 1) & MAX_SEQUENCE;
                // 毫秒内序列溢出
                if (sequence == 0) {
                    // 阻塞到下一个毫秒,获得新的时间戳
                    currentStamp = getNextMill();
                }
            } else {
                //不同毫秒内，序列号置0
                sequence = 0L;
            }
            lastStamp = currentStamp;
            //时间戳部分+机器标识部分+序列号部分
            return (currentStamp - START_TIMESTAMP) << TIMESTAMP_LEFT | machineIdPart | sequence;
        }
        /**
         * 阻塞到下一个毫秒，直到获得新的时间戳
         */
        private long getNextMill() {
            long mill = timeGen();
            //
            while (mill <= lastStamp) {
                mill = timeGen();
            }
            return mill;
        }
        /**
         * 返回以毫秒为单位的当前时间
         */
        protected long timeGen() {
            return System.currentTimeMillis();
        }
    }

    /**
     * 获取long类型雪花ID
     */
    public static long uniqueLong() {
        return SnowIdUtils.SnowFlake.SNOW_FLAKE.nextId();
    }
    /**
     * 获取String类型雪花ID
     */
    public static String uniqueLongHex() {
        return String.format("%016x", uniqueLong());
    }

    /**
     * 测试
     */
    public static void main(String[] args) throws InterruptedException {
        //计时开始时间
        long start = System.currentTimeMillis();
        //让100个线程同时进行
        final CountDownLatch latch = new CountDownLatch(100);
        //判断生成的20万条记录是否有重复记录
        final Map<Long, Integer> map = new ConcurrentHashMap();
        for (int i = 0; i < 100; i++) {
            //创建100个线程
            new Thread(() -> {
                for (int s = 0; s < 2000; s++) {
                    long snowID = SnowIdUtils.uniqueLong();
                    log.info("生成雪花ID={}",snowID);
                    Integer put = map.put(snowID, 1);
                    if (put != null) {
                        throw new RuntimeException("主键重复");
                    }
                }
                latch.countDown();
            }).start();
        }
        //让上面100个线程执行结束后，在走下面输出信息
        latch.await();
        log.info("生成20万条雪花ID总用时={}", System.currentTimeMillis() - start);
    }
}
```

#### 2、测试结果

```shell
16:35:56.814 [Thread-1] INFO  c.k.h.u.SnowIdUtils - [lambda$0,146] - 生成雪花ID=4159236345865
16:35:56.814 [Thread-1] INFO  c.k.h.u.SnowIdUtils - [lambda$0,146] - 生成雪花ID=4159236345866
16:35:56.814 [Thread-1] INFO  c.k.h.u.SnowIdUtils - [lambda$0,146] - 生成雪花ID=4159236345867
16:35:56.814 [Thread-1] INFO  c.k.h.u.SnowIdUtils - [lambda$0,146] - 生成雪花ID=4159236345868
16:35:56.814 [Thread-1] INFO  c.k.h.u.SnowIdUtils - [lambda$0,146] - 生成雪花ID=4159236345869
16:35:56.814 [Thread-1] INFO  c.k.h.u.SnowIdUtils - [lambda$0,146] - 生成雪花ID=4159236345870
16:35:56.814 [Thread-1] INFO  c.k.h.u.SnowIdUtils - [lambda$0,146] - 生成雪花ID=4159236345871
16:35:56.814 [Thread-1] INFO  c.k.h.u.SnowIdUtils - [lambda$0,146] - 生成雪花ID=4159236345872
16:35:56.814 [Thread-1] INFO  c.k.h.u.SnowIdUtils - [lambda$0,146] - 生成雪花ID=4159236345873
16:35:56.814 [Thread-1] INFO  c.k.h.u.SnowIdUtils - [lambda$0,146] - 生成雪花ID=4159236345874
16:35:56.814 [Thread-1] INFO  c.k.h.u.SnowIdUtils - [lambda$0,146] - 生成雪花ID=4159236345875
16:35:56.836 [Thread-1] INFO  c.k.h.u.SnowIdUtils - [lambda$0,146] - 生成雪花ID=4159328620544
16:35:56.836 [Thread-1] INFO  c.k.h.u.SnowIdUtils - [lambda$0,146] - 生成雪花ID=4159328620545
16:35:56.836 [Thread-1] INFO  c.k.h.u.SnowIdUtils - [lambda$0,146] - 生成雪花ID=4159328620546
16:35:56.836 [Thread-1] INFO  c.k.h.u.SnowIdUtils - [lambda$0,146] - 生成雪花ID=4159328620547
16:35:56.836 [Thread-1] INFO  c.k.h.u.SnowIdUtils - [lambda$0,146] - 生成雪花ID=4159328620548
16:35:56.836 [Thread-1] INFO  c.k.h.u.SnowIdUtils - [lambda$0,146] - 生成雪花ID=4159328620549
16:35:56.836 [Thread-1] INFO  c.k.h.u.SnowIdUtils - [lambda$0,146] - 生成雪花ID=4159328620550
16:35:56.836 [Thread-1] INFO  c.k.h.u.SnowIdUtils - [lambda$0,146] - 生成雪花ID=4159328620551
16:35:56.836 [Thread-1] INFO  c.k.h.u.SnowIdUtils - [lambda$0,146] - 生成雪花ID=4159328620552
16:35:56.836 [Thread-1] INFO  c.k.h.u.SnowIdUtils - [lambda$0,146] - 生成雪花ID=4159328620553
16:35:56.836 [Thread-1] INFO  c.k.h.u.SnowIdUtils - [lambda$0,146] - 生成雪花ID=4159328620554
16:35:56.837 [Thread-1] INFO  c.k.h.u.SnowIdUtils - [lambda$0,146] - 生成雪花ID=4159332814848
16:35:56.837 [Thread-1] INFO  c.k.h.u.SnowIdUtils - [lambda$0,146] - 生成雪花ID=4159332814849
16:35:56.837 [Thread-1] INFO  c.k.h.u.SnowIdUtils - [lambda$0,146] - 生成雪花ID=4159332814850
16:35:56.837 [Thread-1] INFO  c.k.h.u.SnowIdUtils - [lambda$0,146] - 生成雪花ID=4159332814851
16:35:56.837 [Thread-1] INFO  c.k.h.u.SnowIdUtils - [lambda$0,146] - 生成雪花ID=4159332814852
16:35:56.837 [Thread-1] INFO  c.k.h.u.SnowIdUtils - [lambda$0,146] - 生成雪花ID=4159332814853
16:35:56.837 [Thread-1] INFO  c.k.h.u.SnowIdUtils - [lambda$0,146] - 生成雪花ID=4159332814854
16:35:56.837 [Thread-1] INFO  c.k.h.u.SnowIdUtils - [lambda$0,146] - 生成雪花ID=4159332814855
16:35:56.837 [Thread-1] INFO  c.k.h.u.SnowIdUtils - [lambda$0,146] - 生成雪花ID=4159332814856
16:35:56.837 [Thread-1] INFO  c.k.h.u.SnowIdUtils - [lambda$0,146] - 生成雪花ID=4159332814857
16:35:56.837 [Thread-1] INFO  c.k.h.u.SnowIdUtils - [lambda$0,146] - 生成雪花ID=4159332814858
16:35:56.837 [Thread-1] INFO  c.k.h.u.SnowIdUtils - [lambda$0,146] - 生成雪花ID=4159332814859
16:35:56.837 [Thread-1] INFO  c.k.h.u.SnowIdUtils - [lambda$0,146] - 生成雪花ID=4159332814860
16:35:56.837 [Thread-1] INFO  c.k.h.u.SnowIdUtils - [lambda$0,146] - 生成雪花ID=4159332814861
16:35:56.837 [main] INFO  c.k.h.u.SnowIdUtils - [main,157] - 生成20万条雪花ID总用时=9628
```

从图中我们可以得出

1、在100个线程并发下，生成20万条雪花ID的时间大概在1.6秒左右，所有所性能还是蛮ok的。

2、生成20万条雪花ID并没有一条相同的ID,如果有的话就会抛出异常了。



#### 3、为什么说41位时间戳最长只能有69年

我们思考41的二进制，最大值也就41位都是1，也就是也就是说41位可以表示![2^{41}-1](https://math.jianshu.com/math?formula=2%5E%7B41%7D-1)个毫秒的值，转化成单位年则是

![(2^{41}-1) / (1000 * 60 * 60 * 24 *365) = 69](https://math.jianshu.com/math?formula=(2%5E%7B41%7D-1)%20%2F%20(1000%20*%2060%20*%2060%20*%2024%20*365)%20%3D%2069)年

所以说雪花算法生成的ID,只能保证69年内不会重复，如果超过69年的话，那就考虑换个服务器部署吧，并且要保证该服务器的ID和之前都没有重复过。

或者可以修改`START_TIMESTAMP`把时间戳做成动态可配置的。如果想做百年企业的话我们可以改造



------

=====================================================================

微信公众号：

![](https://www.codepeople.cn/imges/weixin_icon/weixin.jpg)