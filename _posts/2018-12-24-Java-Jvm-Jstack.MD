---
title:  "怎样使用jstack诊断Java应用程序故障"
categories: jstack
tags: jstack
author: LIUREN
---

# 怎样使用jstack诊断Java应用程序故障

>  最近一段时间,我们的生产系统升级频繁出现故障,具体故障现象是启动后10来分钟就出现交易缓慢,处理线程耗尽等现象,并且故障发生的频率蛮高的。经过详细的诊断和排查,终于发现了问题,是groovy在osgi中运行会出现classloader死锁,最后我们也解决了这个问题。 
> ​        如果单靠通过查看代码是很难去发现这个问题,在这一次故障排查中,我也学到了怎样更好的使用jvm监控工具来进行诊断,主要用到了jstack和jmap命令,jmap上次已经讲过就不再讲了,下面就一个例子来讲怎么使用jstack来对的Java程序进行诊断。 
> ​        首先让我们来了解一下jstack这个命令的作用,jstack 是一个可以返回在应用程序上运行的各种各样线程的一个完整转储的实用程序,您可以使用它查明问题。jstack [-l] <pid>,jpid可以通过使用jps命令来查看当前Java程序的jpid值,-l是可选参数,它可以显示线程阻塞/死锁情况。 



```java
/** 
 * 死锁例子 
 * @author crane.ding 
 * @since 2011-3-20 
 */  
public class DeadLock {  
  
    public static void main(String[] args) {  
        final Object obj_1 = new Object(), obj_2 = new Object();  
          
        Thread t1 = new Thread("t1"){  
            @Override  
            public void run() {  
                synchronized (obj_1) {  
                    try {  
                        Thread.sleep(3000);  
                    } catch (InterruptedException e) {}  
                      
                    synchronized (obj_2) {  
                        System.out.println("thread t1 done.");  
                    }  
                }  
            }  
        };  
          
        Thread t2 = new Thread("t2"){  
            @Override  
            public void run() {  
                synchronized (obj_2) {  
                    try {  
                        Thread.sleep(3000);  
                    } catch (InterruptedException e) {}  
                      
                    synchronized (obj_1) {  
                        System.out.println("thread t2 done.");  
                    }  
                }  
            }  
        };  
          
        t1.start();  
        t2.start();  
    }  
      
}  
```

以上DeadLock类是一个死锁的例子,假使在我们不知情的情况下,运行DeadLock后,发现等了N久都没有在屏幕打印线程完成信息。这个时候我们就可以使用jps查看该程序的jpid值和使用jstack来生产堆栈结果问题。 

```shell
$ java -cp deadlock.jar DeadLock &  
```

然后

```shell
$ jps  
  3076 Jps  
  448 DeadLock  
$ jstack -l 448 > deadlock.jstack  
```

结果文件deadlock.jstack内容如下： 

```xml
2011-03-20 23:05:20  
Full thread dump Java HotSpot(TM) Client VM (19.1-b02 mixed mode, sharing):  
  
"DestroyJavaVM" prio=6 tid=0x00316800 nid=0x9fc waiting on condition [0x00000000]  
   java.lang.Thread.State: RUNNABLE  
  
   Locked ownable synchronizers:  
    - None  
  
"t2" prio=6 tid=0x02bcf000 nid=0xc70 waiting for monitor entry [0x02f6f000]  
   java.lang.Thread.State: BLOCKED (on object monitor)  
    at com.demo.DeadLock$2.run(DeadLock.java:40)  
    - waiting to lock <0x22a297a8> (a java.lang.Object)  
    - locked <0x22a297b0> (a java.lang.Object)  
  
   Locked ownable synchronizers:  
    - None  
  
"t1" prio=6 tid=0x02bce400 nid=0xba0 waiting for monitor entry [0x02f1f000]  
   java.lang.Thread.State: BLOCKED (on object monitor)  
    at com.demo.DeadLock$1.run(DeadLock.java:25)  
    - waiting to lock <0x22a297b0> (a java.lang.Object)  
    - locked <0x22a297a8> (a java.lang.Object)  
  
   Locked ownable synchronizers:  
    - None  
  
"Low Memory Detector" daemon prio=6 tid=0x02bb9400 nid=0xa6c runnable [0x00000000]  
   java.lang.Thread.State: RUNNABLE  
  
   Locked ownable synchronizers:  
    - None  
  
"CompilerThread0" daemon prio=10 tid=0x02bb2800 nid=0xcb8 waiting on condition [0x00000000]  
   java.lang.Thread.State: RUNNABLE  
  
   Locked ownable synchronizers:  
    - None  
  
"Attach Listener" daemon prio=10 tid=0x02bb1000 nid=0x7f4 waiting on condition [0x00000000]  
   java.lang.Thread.State: RUNNABLE  
  
   Locked ownable synchronizers:  
    - None  
  
"Signal Dispatcher" daemon prio=10 tid=0x02bd2800 nid=0xd80 runnable [0x00000000]  
   java.lang.Thread.State: RUNNABLE  
  
   Locked ownable synchronizers:  
    - None  
  
"Finalizer" daemon prio=8 tid=0x02bab000 nid=0xe1c in Object.wait() [0x02d3f000]  
   java.lang.Thread.State: WAITING (on object monitor)  
    at java.lang.Object.wait(Native Method)  
    - waiting on <0x229e1148> (a java.lang.ref.ReferenceQueue$Lock)  
    at java.lang.ref.ReferenceQueue.remove(ReferenceQueue.java:118)  
    - locked <0x229e1148> (a java.lang.ref.ReferenceQueue$Lock)  
    at java.lang.ref.ReferenceQueue.remove(ReferenceQueue.java:134)  
    at java.lang.ref.Finalizer$FinalizerThread.run(Finalizer.java:159)  
  
   Locked ownable synchronizers:  
    - None  
  
"Reference Handler" daemon prio=10 tid=0x02ba6800 nid=0xbe0 in Object.wait() [0x02cef000]  
   java.lang.Thread.State: WAITING (on object monitor)  
    at java.lang.Object.wait(Native Method)  
    - waiting on <0x229e1048> (a java.lang.ref.Reference$Lock)  
    at java.lang.Object.wait(Object.java:485)  
    at java.lang.ref.Reference$ReferenceHandler.run(Reference.java:116)  
    - locked <0x229e1048> (a java.lang.ref.Reference$Lock)  
  
   Locked ownable synchronizers:  
    - None  
  
"VM Thread" prio=10 tid=0x02b6a400 nid=0x568 runnable   
  
"VM Periodic Task Thread" prio=10 tid=0x02bc8400 nid=0x75c waiting on condition   
  
JNI global references: 878  
  
  
Found one Java-level deadlock:  
=============================  
"t2":  
  waiting to lock monitor 0x02baaeec (object 0x22a297a8, a java.lang.Object),  
  which is held by "t1"  
"t1":  
  waiting to lock monitor 0x02baa2bc (object 0x22a297b0, a java.lang.Object),  
  which is held by "t2"  
  
Java stack information for the threads listed above:  
===================================================  
"t2":  
    at com.demo.DeadLock$2.run(DeadLock.java:40)  
    - waiting to lock <0x22a297a8> (a java.lang.Object)  
    - locked <0x22a297b0> (a java.lang.Object)  
"t1":  
    at com.demo.DeadLock$1.run(DeadLock.java:25)  
    - waiting to lock <0x22a297b0> (a java.lang.Object)  
    - locked <0x22a297a8> (a java.lang.Object)  
  
Found 1 deadlock.  
```





> 从这个结果文件我们一看到发现了一个死锁,具体是线程t2在等待线程t1,而线程t1在等待线程t2造成的,同时也记录了线程的堆栈和代码行数,通过这个堆栈和行数我们就可以去检查对应的代码块,从而发现问题和解决问题。 

