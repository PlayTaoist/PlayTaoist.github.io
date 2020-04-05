---
title:  "蚂蚁金服面试题和答案"
categories: Alibaba
tags: Alibaba
author: LIUREN
---

# 蚂蚁金服面试题和答案

#### 1. 自我介绍、自己做的项目和技术领域

#### 2. 项目中的监控：那个监控指标常见的哪些？

| 名词     | 含义                                                         |
| -------- | ------------------------------------------------------------ |
| TPS      | 应用每秒处理的请求数                                         |
| AVG      | 应用对每个请求响应的平均时间                                 |
| TP99     | 99%的请求响应时间小于等于该值                                |
| TP90     | 90%的请求响应时间小于等于该值                                |
| TP50     | 50%的请求响应时间小于等于该值                                |
| FAIL     | 应用对请求响应的成功、失败比率                               |
| 调用链接 | 一次请求所经过的所有系统的集合产生的链条，反馈了系统将的依赖关系及时许 |

##### 2.1服务端监控指标

性能测试通常需要监控的指标包括：

```
服务器 Linux (包括CPU、Memory、Load、I/O)

数据库：MySQL（缓存命中、索引、单条SQL性能、数据库线程数、数据池连接数）

中间件：1.tomcat 2、nginx 3、memcache 4、Redis（包括线程数、连接数、日志）

网络：吞吐量、吞吐率

应用：Jvm内存、日志、Full GC频率
```

##### 2.2客户端监控指标

```
LoadRunner:用户执行情况、场景状态、事物响应时间、TPS、吞吐量

测试机资源：CPU、Memory、网络、磁盘空间
```



#### 3.微服务涉及到的技术以及需要注意的问题有哪些？

```
1.每个微服务都很小，这样能聚焦一个指定的业务功能或业务需求。

2.微服务能够被小团队单独开发，这个小团队是2到5人的开发人员组成。

3.微服务是松耦合的，是有功能意义的服务，无论是在开发阶段或部署阶段都是独立的。

4.微服务能使用不同的语言开发。

5.微服务易于被一个开发人员理解，修改和维护，这样小团队能够更关注自己的工作成果。无需通过合作才能体现价值。
```

##### 3.1微服务架构的缺点

```
1.微服务架构可能带来过多的操作。

2.需要DevOps技巧 (http://en.wikipedia.org/wiki/DevOps)。

3.可能双倍的努力。

4.分布式系统可能复杂难以管理。

5.因为分布部署跟踪问题难。

6.当服务数量增加，管理复杂性增加。
```

#### 4.注册中心你了解了哪些？

`在微服务架构中，注册中心是核心的基础服务之一。在微服务架构流行之前，注册中心就已经开始出现在分布式架构的系统中。Dubbo是一个在国内比较流行的分布式框架，被大量的中小型互联网公司所采用，Dubbo是一个非常实用的框架，提供了比较完善的服务治理功能，而服务治理的实现主要依靠的就是注册中心。`

#### 5.consul 的可靠性你了解吗？

`那么consul是啥？consul就是提供服务发现的工具`

`consul是分布式的、高可用、横向扩展的`

#### 6.consul 的机制你有没有具体深入过？有没有和其他的注册中心对比过？

##### ZooKeeper

`历史悠久，数据存储格式类似文件系统，通过私有协议访问，集群式架构。优点是成熟稳定，缺点是系统复杂，资源占用高`

##### etcd

`etcd是通过HTTP协议访问的k/v存储系统，采用集群架构，容易部署和使用。但他更多功能是提供存储，要实现服务发现还得配合一些第三方的应用或者自己实现。 
\* “registrator”, 自动注册工具，将服务提供方的信息存储到etcd, consul这种kv存储系统 * ”confd“，轻量级的配置管理工具，他可以从etcd里取最新的服务信息生成配置文件，服务使用方就可以用它来实时更新配置文件`

##### Consul

`Consul 提供了高可用的kv存储，集群架构，这点和etcd zookeeper类似。 另外也提供了自动服务发现注册的套件，并且能否对服务进行健康检查。 结合consul-template可以实现服务提供方信息更新(比如增加了API服务器)时，自动生成配置文件给服务使用方自动更新配置。`

#### 7.项目用 Spring 比较多，有没有了解 Spring 的原理？AOP 和 IOC 的原理

`Spring的两个核心概念是IOC（控制反转）和AOP（面向切面编程）`

[**spring原理**](https://www.jianshu.com/p/c403609185a5 "spring原理")

#### 8.Spring Boot除了自动配置，相比传统的 Spring 有什么其他的区别？

`从本质上来说，Spring Boot就是Spring,它做了那些没有它你也会去做的Spring Bean配置`

```
SpringBoot的优点？Spring由于其繁琐的配置，一度被人认为“配置地狱”，各种XML、Annotation配置，让人眼花缭乱，而且如果出错了也很难找出原因。SpringBoot帮助开发者快速启动一个Web容器；SpringBoot继承了原有Spring框架的优秀基因；SpringBoot简化了使用Spring的过程。
SpringBoot的缺点？Spring Boot作为一个微框架，离微服务的实现还是有距离的。没有提供相应的服务发现和注册的配套功能，自身的acturator所提供的监控功能，也需要与现有的监控对接。没有配套的安全管控方案，对于REST的落地，还需要自行结合实际进行URI的规范化工作。
```

#### 9.Spring Cloud 有了解多少？



#### 10.Spring Bean 的生命周期

`实例化`==>`填充属性`==>`调用BeanNameAware的setBeanName方法`==>`调用BeanFactoryAware的setBeanDactory方法`==>`调用ApplicationContext方法`==>`调用BeanPostProcess的postProcessBeforeInitialization方法`==>`调用InitializingBean的afterPropertiesSet方法`==>`调用定制的初始化方法`==>`调用BeanPostProcess的postProcessAfterInitialization方法`==>`Bean准备就绪`==>`调用DispostbleBean的destory方法`==>`调用定制的销毁方法`

```
Spring 只帮我们管理单例模式 Bean 的**完整**生命周期，对于 prototype 的 bean ，Spring 在创建好交给使用者之后则不会再管理后续的生命周期。
```



#### 11.HashMap 和 hashTable 区别？

`HashMap线程是不安全的，HashTable是安全的。HashTable的key和value都不能为null，HashMap的key和value可以为null`

#### 12.Object 的 hashcode 方法重写了，equals 方法要不要改？

 个人观点：`如果我们重写了equals方法的话，我们就必须重写hashcode方法，为什么equals()相等，那么hashCode()必须相等。因为，如果两个对象的equals()方法返回true，则它们在哈希表中只应该出现一次；如果hashCode()不相等，那么它们会被散列到表中不同的位置，哈希表中不止出现一次`

`如果只修改了hashcode的方法，equlas方法可以不必要修改`

#### 13.Hashmap 线程不安全的出现场景

```
我们都知道HashMap初始容量大小为16,一般来说，当有数据要插入时，都会检查容量有没有超过设定的thredhold，如果超过，需要增大Hash表的尺寸，但是这样一来，整个Hash表里的元素都需要被重算一遍。这叫rehash
```

```
外一个比较明显的线程不安全的问题是HashMap的get操作可能因为resize而引起死循环（cpu100%）
```

#### 14.线上服务 CPU 很高该怎么做？有哪些措施可以找到问题

```
top oder by with P：1040 // 首先按进程负载排序找到  axLoad(pid)
top -Hp 进程PID：1073    // 找到相关负载 线程PID
printf “0x%x\n”线程PID： 0x431  // 将线程PID转换为 16进制，为后面查找 jstack 日志做准备
jstack  进程PID | vim +/十六进制线程PID -        // 例如：jstack 1040|vim +/0x431 -
```

[记一次线上Java程序导致服务器CPU占用率过高的问题排除过程](https://blog.csdn.net/u013991521/article/details/52781423)

[线上服务 CPU 100%？一键定位不骗你-掘金](https://juejin.im/entry/59a305c351882524417ad43b)

#### 15.JDK 中有哪几个线程池？顺带把线程池讲了个遍

##### java封装的几个线程池介绍

```
FixedThreadPool:
FixedThreadPool并不是一个类，它是由Executors工具类创建出来的一个固定线程数的ThreadPoolEexcutor的对象。
SingleThreadPool:
SingleThreadPool是只有一个线程的线程池，内部实现和FixedThreadPool一样，不过就是线程数有区别。
CachedThreadPool:
一个可缓存线程池，如果线程池长度超过处理需要，可灵活回收空闲线程，若无可回收，则新建线程
ScheduledThreadPoolExecutor:
一个定长线程池，支持定时及周期性任务执行。
```

#### 16.SQL 优化的常见方法有哪些

`1.应尽量避免在 where 子句中使用!=或<>操作符，否则将引擎放弃使用索引而进行全表扫描`

`2.对查询进行优化，应尽量避免全表扫描，首先应考虑在 where 及 order by 涉及的列上建立索引`

`3.应尽量避免在 where 子句中对字段进行 null值判断，否则将导致引擎放弃使用索引而进行全表扫描，`

​	`如：select id from t where num is null可以在num上设置默认值0，确保表中num列没有null值，`

​	`然后这样查询：select id from t where num=0`

`4.尽量避免在 where 子句中使用 or 来连接条件，否则将导致引擎放弃使用索引而进行全表扫描，`

​	`如：select id from t where num=10 or num=20 `

​	`可以这样查询：select id from t where num=10 unionall select id from t where num=20`

`5.下面的查询也将导致全表扫描 select id from t where name like ‘%c%’,左模糊和全模糊查询都会导致搜索引擎放弃索引直接全表扫描，要想使用索引可以使用右模糊查询`

`6.in 和 not in 也要慎用，否则会导致全表扫描 `

​	`如：select id from t where num in(1,2,3) `

​	`对于连续的数值，能用 between 就不要用 in 了：select id from t where num between 1 and 3`

`7.如果在 where 子句中使用参数，也会导致全表扫描。因为SQL只有在运行时才会解析局部变量，但优化程序不能将访问计划的选择推迟到运行时；它必须在编译时进行选择。然而，如果在编译时建立访问计划，变量的值还是未知的，因而无法作为索引选择的输入项。`

​	`如下面语句将进行全表扫描：select id from t where num = @num`

​	`可以改为强制查询使用索引：select id from t with(index(索引名)) where num = @num`

`8.应尽量避免在 where 子句中对字段进行表达式操作，这将导致引擎放弃使用索引而进行全表扫描。`

​	`如：select id from t where num/2=100 `

​	`应改为: select id from t where num=100*2`

`9.应尽量避免在where子句中对字段进行函数操作，这将导致引擎放弃使用索引而进行全表扫描。`

​	`如：select id from t where substring(name,1,3)=’abc’–name以abc开头的id `

​	`select id from t where date diff(day,createdate,’2005-11-30′)=0–’2005-11-30′生成的id`

​	`应修改为：select id from t where name like‘abc%’select id from t where 					createdate>=’2005-11-30′ andcreatedate<’2005-12-1′`

`10.不要在 where 子句中的“=”左边进行函数、算术运算或其他表达式运算，否则系统将可能无法正确使用索引`

`11.在使用索引字段作为条件时，如果该索引是复合索引，那么必须使用到该索引中的第一个字段作为条件时才能保证系统使用该索引，否则该索引将不会被使用，并且应尽可能的让字段顺序与索引顺序相一致`

`12.不要写一些没有意义的查询，如需要生成一个空表结构：`

​	`如：select col1,col2 into #t from t where 1=0这类代码不会返回任何结果集，但是会消耗系统资源的，`

​	`应改成这样：create table #t(…)`

`13.很多时候用 exists 代替 in 是一个好的选择 `

​	`如：select num from a where num in(select num from b)  `

​	`修改成 select num from a where exists(select 1 from b where num=a.num)`

`14.并不是所有索引对查询都有效，SQL是根据表中数据来进行查询优化的，当索引列有大量数据重复时，SQL查询可能不会去利用索引，如一表中有字段 sex，male、female几乎各一半，那么即使在sex上建了索引也对查询效率起不了作用。`

`15.索引并不是越多越好，索引固然可以提高相应的 select 的效率，但同时也降低了 insert 及 update 的效率，因为 insert 或 update 时有可能会重建索引，所以怎样建索引需要慎重考虑，视具体情况而定。一个表的索引数最好不要超过6个，若太多则应考虑一些不常使用到的列上建的索引是否有必要`

`16.尽量使用数字型字段，若只含数值信息的字段尽量不要设计为字符型，这会降低查询和连接的性能，并会增加存储开销。这是因为引擎在处理查询和连接时会逐个比较字符串中每一个字符，而对于数字型而言只需要比较一次就够了`

`17.尽可能的使用varchar/nvarchar 代替 char/nchar ，因为首先变长字段存储空间小，可以节省存储空间，其次对于查询来说，在一个相对较小的字段内搜索效率显然要高些。`

`18.任何地方都不要使用 select *from t ，用具体的字段列表代替“*”，不要返回用不到的任何字段。`

`19.尽量避免使用游标，因为游标的效率较差，如果游标操作的数据超过1万行，那么就应该考虑改写`

`20.在所有的存储过程和触发器的开始处设置SET NOCOUNT ON ，在结束时设置 SET NOCOUNT OFF 。无需在执行存储过程和触发器的每个语句后向客户端发送 DONE_IN_PROC 消息。`

`21.尽量避免向客户端返回大数据量，若数据量过大，应该考虑相应需求是否合理。`



#### 17.SQL 索引的顺序，字段的顺序

`遵循最左原则`

#### 18.查看 SQL 是不是使用了索引？（有什么工具）

`查看SQL语句是否引用了索引，可以使用EXPLAIN 方法如：EXPLAIN SELECT * FROM student WHERE   cid=1;`

以前有个工具叫做mysqlreport。现在他的身份就是[percona toolkit](https://www.percona.com/software/database-tools/percona-toolkit)。点击链接查看官方网站。可以[下载使用手册](https://learn.percona.com/download-percona-tollkit-2-2-manual)。

#### 19.TCP 和 UDP 的区别？TCP 数据传输过程中怎么做到可靠的？

```
TCP与UDP区别总结：
1、TCP面向连接（如打电话要先拨号建立连接）;UDP是无连接的，即发送数据之前不需要建立连接
2、TCP提供可靠的服务。也就是说，通过TCP连接传送的数据，无差错，不丢失，不重复，且按序到达;UDP尽最大努力交付，即不保   证可靠交付
3、TCP面向字节流，实际上是TCP把数据看成一连串无结构的字节流;UDP是面向报文的
  UDP没有拥塞控制，因此网络出现拥塞不会使源主机的发送速率降低（对实时应用很有用，如IP电话，实时视频会议等）
4、每一条TCP连接只能是点到点的;UDP支持一对一，一对多，多对一和多对多的交互通信
5、TCP首部开销20字节;UDP的首部开销小，只有8个字节
6、TCP的逻辑通信信道是全双工的可靠信道，UDP则是不可靠信道
```

```
TCP协议与UDP协议的区别
首先咱们弄清楚，TCP协议和UCP协议与TCP/IP协议的联系，很多人犯糊涂了，
一直都是说TCP协议与UDP协议的区别，我觉得这是没有从本质上弄清楚网络通信！
TCP/IP协议是一个协议簇。里面包括很多协议的，UDP只是其中的一个， 之所以命名为TCP/IP协议，因为TCP、IP协议是两个很重要的协议，就用他两命名了。

TCP/IP协议集包括应用层,传输层，网络层，网络访问层。

其中应用层包括:
1、超文本传输协议（HTTP）:万维网的基本协议；
2、文件传输（TFTP简单文件传输协议）；
3、远程登录（Telnet），提供远程访问其它主机功能, 它允许用户登录internet主机，并在这台主机上执行命令；
4、网络管理（SNMP简单网络管理协议），该协议提供了监控网络设备的方法， 以及配置管理,统计信息收集,性能管理及安全管理等；
5、域名系统（DNS），该系统用于在internet中将域名及其公共广播的网络节点转换成IP地址。

其次网络层包括:
1、Internet协议（IP）；
2、Internet控制信息协议（ICMP）；
3、地址解析协议（ARP）；
4、反向地址解析协议（RARP）。

最后说网络访问层:
网络访问层又称作主机到网络层（host-to-network），网络访问层的功能包括IP地址与物理地址硬件的映射， 以及将IP封装成帧.基于不同硬件类型的网络接口，网络访问层定义了和物理介质的连接. 当然我这里说得不够完善，TCP/IP协议本来就是一门学问，每一个分支都是一个很复杂的流程， 但我相信每位学习软件开发的同学都有必要去仔细了解一番。

下面着重讲解一下TCP协议和UDP协议的区别:
TCP三次握手过程
第一次握手：主机A通过向主机B 发送一个含有同步序列号的标志位的数据段给主机B，向主机B 请求建立连接，通过这个数据段， 主机A告诉主机B 两件事：我想要和你通信；你可以用哪个序列号作为起始数据段来回应我。

第二次握手：主机B 收到主机A的请求后，用一个带有确认应答（ACK）和同步序列号（SYN）标志位的数据段响应主机A，也告诉主机A两件事：我已经收到你的请求了，你可以传输数据了；你要用那个序列号作为起始数据段来回应我

第三次握手：主机A收到这个数据段后，再发送一个确认应答，确认已收到主机B 的数据段："我已收到回复，我现在要开始传输实际数据了，这样3次握手就完成了，主机A和主机B 就可以传输数据了。

3次握手的特点
没有应用层的数据 SYN这个标志位只有在TCP建立连接时才会被置1 握手完成后SYN标志位被置0。

TCP建立连接要进行3次握手，而断开连接要进行4次
第一次： 当主机A完成数据传输后,将控制位FIN置1，提出停止TCP连接的请求 ；

第二次： 主机B收到FIN后对其作出响应，确认这一方向上的TCP连接将关闭,将ACK置1；

第三次： 由B 端再提出反方向的关闭请求,将FIN置1 ；

第四次： 主机A对主机B的请求进行确认，将ACK置1，双方向的关闭结束.
名词解释
1、ACK 是TCP报头的控制位之一，对数据进行确认。确认由目的端发出， 用它来告诉发送端这个序列号之前的数据段都收到了。 比如确认号为X，则表示前X-1个数据段都收到了，只有当ACK=1时,确认号才有效，当ACK=0时，确认号无效，这时会要求重传数据，保证数据的完整性。

2、SYN 同步序列号，TCP建立连接时将这个位置1。

3、FIN 发送端完成发送任务位，当TCP完成数据传输需要断开时,，提出断开连接的一方将这位置1。

TCP的包头结构：
源端口 16位；

目标端口 16位；

序列号 32位；

回应序号 32位；

TCP头长度 4位；

reserved 6位；

控制代码 6位；

窗口大小 16位；

偏移量 16位；

校验和 16位；

选项 32位(可选)；

这样我们得出了TCP包头的最小长度，为20字节。

UDP（User Data Protocol，用户数据报协议）
1、UDP是一个非连接的协议，传输数据之前源端和终端不建立连接， 当它想传送时就简单地去抓取来自应用程序的数据，并尽可能快地把它扔到网络上。 在发送端，UDP传送数据的速度仅仅是受应用程序生成数据的速度、 计算机的能力和传输带宽的限制； 在接收端，UDP把每个消息段放在队列中，应用程序每次从队列中读一个消息段。

2、 由于传输数据不建立连接，因此也就不需要维护连接状态，包括收发状态等， 因此一台服务机可同时向多个客户机传输相同的消息。

3、UDP信息包的标题很短，只有8个字节，相对于TCP的20个字节信息包的额外开销很小。

4、吞吐量不受拥挤控制算法的调节，只受应用软件生成数据的速率、传输带宽、 源端和终端主机性能的限制。

5、UDP使用尽最大努力交付，即不保证可靠交付， 因此主机不需要维持复杂的链接状态表（这里面有许多参数）。

6、UDP是面向报文的。发送方的UDP对应用程序交下来的报文， 在添加首部后就向下交付给IP层。既不拆分，也不合并，而是保留这些报文的边界， 因此，应用程序需要选择合适的报文大小。



我们经常使用“ping”命令来测试两台主机之间TCP/IP通信是否正常， 其实“ping”命令的原理就是向对方主机发送UDP数据包，然后对方主机确认收到数据包， 如果数据包是否到达的消息及时反馈回来，那么网络就是通的。



ping命令是用来探测主机到主机之间是否可通信，如果不能ping到某台主机，表明不能和这台主机建立连接。ping命令是使用 IP 和网络控制信息协议 (ICMP)，因而没有涉及到任何传输协议(UDP/TCP) 和应用程序。它发送icmp回送请求消息给目的主机。

ICMP协议规定：目的主机必须返回ICMP回送应答消息给源主机。如果源主机在一定时间内收到应答，则认为主机可达。

UDP的包头结构：
源端口 16位 目的端口 16位 长度 16位 校验和 16位
```

#### 20.说下你知道的排序算法吧

[排序算法](https://juejin.im/post/5b9492def265da0aff171b94)

[面试中的排序算法总结(https://www.cnblogs.com/wxisme/p/5243631.html)](https://www.cnblogs.com/wxisme/p/5243631.html)

​	**冒泡排序**、**选择排序**、**插入排序**、**快速排序**、**堆排序**、**希尔排序**、**归并排序**、**计数排序**、**桶排序**、**基数排序**、

| 算法     | 最快时间复杂度 | 平均时间复杂度 | 最坏时间复杂度 | 空间复杂度 | 是否稳定 |
| -------- | -------------- | -------------- | -------------- | ---------- | -------- |
| 冒泡排序 | Ω(n)           | Θ(n2)          | O(n2)          | O(1)       | 稳定     |
| 插入排序 | Ω(n)           | Θ(n2)          | O(n2)          | O(1)       | 稳定     |
| 希尔排序 | Ω(nlogn)       | Θ(n(log(n))2)  | O(n(log(n))2)  | O(1)       | 不稳定   |
| 选择排序 | Ω(n2)          | Θ(n2)          | O(n2)          | O(1)       | 不稳定   |
| 堆排序   | Ω(nlogn)       | Θ(nlogn)       | O(nlogn)       | O(1)       | 不稳定   |
| 归并排序 | Ω(nlogn)       | Θ(nlogn)       | O(nlogn)       | O(n)       | 稳定     |
| 快速排序 | Ω(nlogn)       | Θ(nlogn)       | O(nlogn)       | O(logn)    | 不稳定   |
| 基数排序 | Ω(n+b)         | Θ(n+b)         | O(n+b)         | O(n+k)     | 稳定     |

> O表示上界(小于等于)Ω表示下界(大于等于)Θ表示即是上界也是下界(等于)

#### 21.查找一个数组的中位数？

[【算法】无序数组中求中位数(https://blog.csdn.net/u010983881/article/details/78160671)](https://blog.csdn.net/u010983881/article/details/78160671)

#### 22.http 默认端口，https 默认端口

```
⑴. HTTP协议代理服务器常用端口号：80/8080/3128/8081/9080
⑵. SOCKS代理协议服务器常用端口号：1080
⑶. FTP（文件传输）协议代理服务器常用端口号：21
⑷. Telnet（远程登录）协议代理服务器常用端口：23

HTTP服务器，默认的端口号为80/tcp（木马Executor开放此端口）；
HTTPS（securely transferring web pages）服务器，默认的端口号为443/tcp 443/udp；
Telnet（不安全的文本传送），默认端口号为23/tcp（木马Tiny Telnet Server所开放的端口）；
FTP，默认的端口号为21/tcp（木马Doly Trojan、Fore、Invisible FTP、WebEx、WinCrash和Blade Runner所开放的端口）；
TFTP（Trivial File Transfer Protocol），默认的端口号为69/udp；
SSH（安全登录）、SCP（文件传输）、端口重定向，默认的端口号为22/tcp；
SMTP Simple Mail Transfer Protocol (E-mail），默认的端口号为25/tcp（木马Antigen、Email Password Sender、Haebu Coceda、Shtrilitz Stealth、WinPC、WinSpy都开放这个端口）；
POP3 Post Office Protocol (E-mail) ，默认的端口号为110/tcp；
WebLogic，默认的端口号为7001；
Webshpere应用程序，默认的端口号为9080；
webshpere管理工具，默认的端口号为9090；
JBOSS，默认的端口号为8080；
TOMCAT，默认的端口号为8080；
WIN2003远程登陆，默认的端口号为3389；
Symantec AV/Filter for MSE,默认端口号为 8081；
Oracle 数据库，默认的端口号为1521；
ORACLE EMCTL，默认的端口号为1158；
Oracle XDB（XML 数据库），默认的端口号为8080；
Oracle XDB FTP服务，默认的端口号为2100；
MS SQL*SERVER数据库server，默认的端口号为1433/tcp 1433/udp；
MS SQL*SERVER数据库monitor，默认的端口号为1434/tcp 1434/udp；
```

23.DNS 你知道是干嘛的吗

```
DNS是Domain Name Service的缩写，翻译过来就是计算机域名服务器（也有扩写成Domain Name System，译为计算机域名系统）。而之所以本文称DNS服务器为“翻译官”，是因为DNS是进行域名(domain name)和与之相对应的IP地址(IP address)转换的服务器。
一个域名可以指向多个ip，用来做负载均衡嘛。
同样一个ip可以被多个域名指向，就是大家所购买的虚拟主机嘛;一个域名至少解析到一个IP地址，可以解析到多个个IP地址，DNS轮询和CDN加速就是这个原理。
```

24.git rebase 和 merge 有什么区别



```
marge 特点：自动创建一个新的commit
如果合并的时候遇到冲突，仅需要修改后重新commit
优点：记录了真实的commit情况，包括每个分支的详情
缺点：因为每次merge会自动产生一个merge commit，所以在使用一些git 的GUI tools，特别是commit比较频繁时，看到分支很杂乱。

rebase 特点：会合并之前的commit历史
优点：得到更简洁的项目历史，去掉了merge commit
缺点：如果合并出现代码问题不容易定位，因为re-write了history
```

#### 25.常用的负载均衡，该怎么用

[简单介绍几种常用的负载均衡原理(https://www.jianshu.com/p/da6e562fa3a6)](https://www.jianshu.com/p/da6e562fa3a6)

[常用的负载均衡软件详解(https://blog.csdn.net/chengxuyuanyonghu/article/details/78500297)](https://blog.csdn.net/chengxuyuanyonghu/article/details/78500297)

[负载均衡的几种常用算法(https://www.cnblogs.com/me115/p/5000465.html)](https://www.cnblogs.com/me115/p/5000465.html)



#### 26.网关能够为后端服务带来哪些好处



#### 27.hashcode 和 equals 方法常用地方

Java的基类Object提供了一些方法，其中equals()方法用于判断两个对象是否相等，hashCode()方法用于计算对象的哈希码。equals()和hashCode()都不是final方法，都可以被重写(overwrite)。

#### 28.hashmap put 方法存放的时候怎么判断是否是重复的

Map中判断key是否相同是通过containsKey()方法进行的，它就是先判断key的hashCode是否相同，再判断key是否相等或equals的。如果存放的key值重复那么会直接覆盖掉与原来的Key值

#### 29.ArrayList 和 LinkedList 区别

```
1.ArrayList是实现了基于动态数组的数据结构，每个元素在内存中存储地址是连续的；LinkedList基于链表的数据结构
2.LinkedList查询用的遍历，AyyayList查询用的是数组下标，所以对于查询ArrayList性能高于LinkedList，所以检索性能显然高于通过for循环来查找每个元素的LinkedList。
3.元素插入删除的效率对比，要视插入删除的位置来分析，各有优劣：在列表首位添加（删除）元素，LnkedList性能远远优于ArrayList；在列表中间位置添加（删除）元素，总的来说位置靠前则LnkedList性能优于ArrayList，靠后则相反；在列表末尾位置添加（删除）元素，性能相差不大。
```



#### 30.Set 存的顺序是有序的吗？

```
   我们经常听说List是有序且可重复的，Set是无序且不重复的。这是一个误区，这里所说的顺序有两个概念，一是按照添加的顺序排列，二是按，照自然顺序a-z排列。Set并不是无序的传统所说的Set无序指的是HashSet，它不能保证元素的添加顺序，更不能保证自然顺序，而Set的其他实现类是可以实现这两种顺序的。

1，LinkedHashset : 保证元素添加的自然顺序

2，TreeSet : 保证元素的自然顺序
```



#### 31.HashSet 是不是线程安全的？为什么不是线程安全的？

`hashset其实就是用hashmap实现的，所以是不安全的` 

#### 32.Java 中有哪些线程安全的 Map

```
Java中平时用的最多的Map集合就是HashMap了，它是线程不安全的。
看下面两个场景：
1、当用在方法内的局部变量时，局部变量属于当前线程级别的变量，其他线程访问不了，所以这时也不存在线程安全不安全的问题了。
2、当用在单例对象成员变量的时候呢？这时候多个线程过来访问的就是同一个HashMap了，对同个HashMap操作这时候就存在线程安全的问题了。

HashTable
HashTable的get/put方法都被synchronized关键字修饰，说明它们是方法级别阻塞的，它们占用共享资源锁，所以导致同时只能一个线程操作get或者put，而且get/put操作不能同时执行，所以这种同步的集合效率非常低，一般不建议使用这个集合。
SynchronizedMap
这种是直接使用工具类里面的方法创建SynchronizedMap，把传入进行的HashMap对象进行了包装同步而已，这个同步方式实现也比较简单，看出SynchronizedMap的实现方式是加了个对象锁，每次对HashMap的操作都要先获取这个mutex的对象锁才能进入，所以性能也不会比HashTable好到哪里去，也不建议使用。
ConcurrentHashMap - 推荐
这个也是最推荐使用的线程安全的Map，也是实现方式最复杂的一个集合，每个版本的实现方式也不一样，在jdk8之前是使用分段加锁的一个方式，分成16个桶，每次只加锁其中一个桶，而在jdk8又加入了红黑树和CAS算法来实现。
作者：Java技术栈
链接：https://www.jianshu.com/p/533bb7cf8901
來源：简书
简书著作权归作者所有，任何形式的转载都请联系作者获得授权并注明出处。
```



#### 33.Concurrenthashmap 是怎么做到线程安全的？

```
分段机制：segment，每段加reentrantLock可重入锁
定位元素：1 找segment数组下标  2 找segment的HashEntry数组下标
get方法：不需要加锁，value值使用了volatile关键字修饰
put方法：hash计算段---锁定段---hash计算HashEntry数组---若超多阈值---扩容---释放；
ConcurrentHashMap是线程安全的，那是在他们的内部操作，其外部操作还是需要自己来保证其同步的，特别是静态的ConcurrentHashMap,其有更新和查询的过程，要保证其线程安全，需要syn一个不可变的参数才能保证其原子性
```



#### 34.你项目除了写 Java 代码，还有前端代码，那你知道前端有哪些框架吗

`vue`、`React`、`Angular`：是前段三大主流框架

#### 35.MySQL 分页查询语句

```mysql
mysql> SELECT * FROM table LIMIT 5,10; // 检索记录行 6-15  
  
//为了检索从某一个偏移量到记录集的结束所有的记录行，可以指定第二个参数为 -1：   
mysql> SELECT * FROM table LIMIT 95,-1; // 检索记录行 96-last.  
  
//如果只给定一个参数，它表示返回最大的记录行数目：   
mysql> SELECT * FROM table LIMIT 5; //检索前 5 个记录行  
  
//换句话说，LIMIT n 等价于 LIMIT 0,n。  
```



#### 36.不可重复读会出现在什么场景？

##### 事务不考虑隔离性可能会引发的问题

```
1、脏读
　　　　　脏读指一个事务读取了另外一个事务未提交的数据。

　　　　　这是非常危险的，假设Ａ向Ｂ转帐100元，对应sql语句如下所示
    　　　　　　1.update account set money=money+100 where name='B';    
    　　　　　　2.update account set money=money-100  where name='A';
    　　　 当第1条sql执行完，第2条还没执行(A未提交时)，如果此时Ｂ查询自己的帐户，就会发现自己多了100元钱。如果A等B走后再回滚，B就会损失100元。
```

```
2、不可重复读
　　不可重复读指在一个事务内读取表中的某一行数据，多次读取结果不同。
　　例如银行想查询A帐户余额，第一次查询A帐户为200元，此时A向帐户内存了100元并提交了，银行接着又进行了一次查询，此时A帐户为300元了。银行两次查询不一致，可能就会很困惑，不知道哪次查询是准的。
　　不可重复读和脏读的区别是，脏读是读取前一事务未提交的脏数据，不可重复读是重新读取了前一事务已提交的数据。
　　很多人认为这种情况就对了，无须困惑，当然是后面的为准。我们可以考虑这样一种情况，比如银行程序需要将查询结果分别输出到电脑屏幕和写到文件中，结果在一个事务中针对输出的目的地，进行的两次查询不一致，导致文件和屏幕中的结果不一致，银行工作人员就不知道以哪个为准了。
```

```
3、虚读(幻读)
　　虚读(幻读)是指在一个事务内读取到了别的事务插入的数据，导致前后读取不一致。
　　如丙存款100元未提交，这时银行做报表统计account表中所有用户的总额为500元，然后丙提交了，这时银行再统计发现帐户为600元了，造成虚读同样会使银行不知所措，到底以哪个为准。
```



#### 37.前端浏览器地址的一个 http 请求到后端整个流程是怎么样？能够说下吗？

```
域名解析 --> 发起TCP的3次握手 --> 建立TCP连接后发起http请求 --> 服务器响应http请求，浏览器得到html代码 --> 浏览器解析html代码，并请求html代码中的资源（如js、css、图片等） --> 浏览器对页面进行渲染呈现给用户
```

http请求方法：

```
GET: 完整请求一个资源 （常用）
HEAD: 仅请求响应首部
POST：提交表单  （常用）
PUT: (webdav) 上传文件（但是浏览器不支持该方法）
DELETE：(webdav) 删除
OPTIONS：返回请求的资源所支持的方法的方法
TRACE: 追求一个资源请求中间所经过的代理（该方法不能由浏览器发出）
```

[【一次完整的Http请求过程】(https://blog.csdn.net/zjkC050818/article/details/78345819)](https://blog.csdn.net/zjkC050818/article/details/78345819)

`HTTP协议是无状态的，我们看到查到的用到的返回404，500,200，201,202,301.这些不是HTTP协议的状态码。是HTTP的状态码，就是HTTP请求服务器返回的状态码。HTTP协议和HTTP请求返回状态码是二回事。`



`**HTTP请求方法并不是只有GET和POST，只是最常用的。据RFC2616标准（现行的HTTP/1.1）得知，通常有以下8种方法：OPTIONS、GET、HEAD、POST、PUT、DELETE、TRACE和CONNECT。**`

| Code | HTTP Operation            | Body Contents  | Description                        |
| ---- | ------------------------- | -------------- | ---------------------------------- |
| 200  | GET,PUT                   | 资源           | 操作成功                           |
| 201  | POST                      | 在资源,元数据  | 对象创建成功                       |
| 202  | POST,PUT,DELETE.PATCH     | N/A            | 全请求已经被接受                   |
| 204  | DELETE,PUT,PATCH          | N/A            | 操作已经执行成功，但是没有返回数据 |
| 301  | GET                       | link           | 资源已被移除                       |
| 303  | GET                       | link           | 重定向                             |
| 304  | GET                       | N/A            | 资源没有被修改                     |
| 400  | GET,POST,PUT,DELETE.PATCH | 错误提示(消息) | 参数列表错误(缺少,可是不匹配)      |
| 401  | GET,POST,PUT,DELETE.PATCH | 错误提示(消息) | 未授权                             |
| 403  | GET,POST,PUT,DELETE.PATCH | 错误提示(消息) | 和访问受限，授权过期               |
| 404  | GET,POST,PUT,DELETE.PATCH | 错误提示(消息) | 资源，服务未找到                   |
| 405  | GET,POST,PUT,DELETE.PATCH | 错误提示(消息) | 不允许的http方法                   |
| 409  | GET,POST,PUT,DELETE.PATCH | 错误提示(消息) | 资源冲突，或者资源被锁定           |
| 415  | GET,POST,PUT,DELETE.PATCH | 错误提示(消息) | 不支持的数据(媒体)类型             |
| 429  | GET,POST,PUT,DELETE.PATCH | 错误提示(消息) | 请求过多被限制                     |
| 500  | GET,POST,PUT,DELETE.PATCH | 错误提示(消息) | 系统内部错误                       |
| 501  | GET,POST,PUT,DELETE.PATCH | 错误提示(消息) | 接口未实现                         |



Java中HTTP的CRUD方法

| HTTP方法 | 数据处理 | 说明                                               |
| -------- | -------- | -------------------------------------------------- |
| POST     | Create   | 新增一个没有id的资源                               |
| GET      | Read     | 取得一个资源                                       |
| PUT      | Update   | 更新一个资源。或新增一个含 id 资源(如果 id 不存在) |
| DELETE   | Delete   | 删除一个资源                                       |

