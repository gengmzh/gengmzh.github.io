---
layout: post
title: JVM监控工具
category : jdk
tags : [jvm, monitoring]
---
{% include JB/setup %}

`JVM Monitoring Tools`

Published by gmz on 七月 21st, 2011 - in JAVA  
Sun Hotspot JDK自带的VM监控工具  

## 1. jps 查看JAVA进程状态
jps [options] [hostid]
示例：

	[work@a132 bin]$ jps
	3686 Bootstrap
	5414 Jps


jps会列出系统中所有运行中的JAVA虚拟机  
JAVA虚拟机有三种含义：抽象规范、一个具体实现、一个运行中的虚拟机实例，此处是指虚拟机实例。  
默认情况下输出两列结果，第一列为JVM实例ID，即进程ID，第二列为启动类或jar包shortname，或传给main方法的参数  

选项：
	
	-q 禁止输出上述的第二列
	-m 输出启动时传给main方法的参数
	-l 输出启动类的全限定名，或jar的完整路径
	-v 输出传递给JVM的参数，即JVM启动参数
	-V 输出通过flags file传递给JVM的参数 ？？

示例：
	
	[gmz@server01 ~]$ jps -lmv
	18560 sun.tools.jps.Jps -lmv -Denv.class.path=.:/home/gmz/local/jdk1.6.0_25/lib -Dapplication.home=/home/gmz/local/jdk1.6.0_25 -Xms8m
	18077 org.apache.catalina.startup.Bootstrap start -Djava.util.logging.config.file=/home/gmz/local/tomcat-5.5.33/conf/logging.properties -Xms2000m -Xmx2000m -Xmn800m -XX:PermSize=64m -XX:MaxPermSize=128m -XX:SurvivorRatio=4 -Xss256K -verbose:gc -Xloggc:/logs/gc.log -Djava.awt.headless=true -Djava.util.logging.manager=org.apache.juli.ClassLoaderLogManager -Djava.endorsed.dirs=/home/gmz/local/tomcat-5.5.33/common/endorsed -Dcatalina.base=/home/gmz/local/tomcat-5.5.33 -Dcatalina.home=/home/gmz/local/tomcat-5.5.33 -Djava.io.tmpdir=/home/gmz/local/tomcat-5.5.33/temp
	6174 slave.jar

共列出三个JVM实例，第一个为jps自身，第二个是tomcat，第三个是hudson的slave

## 2. jstat JVM统计监控
jstat [ generalOption | outputOptions vmid [interval[s|ms] [count]] ]  
示例：

	[gmz@server01 ~]$ jstat -gcutil 18077 2000 3
	S0 S1 E O P YGC YGCT FGC FGCT GCT
	0.00 0.00 10.26 1.36 56.37 3 0.054 2 0.520 0.574
	0.00 0.00 10.26 1.36 56.37 3 0.054 2 0.520 0.574
	0.00 0.00 10.26 1.36 56.37 3 0.054 2 0.520 0.574

每隔2s输出一次本地VMID为18077的JVM垃圾收集统计信息  

generalOption可为：
	
	-help 输出帮助信息
	-options 可统计的选项，如：
	[gmz@server01 ~]$ jstat -options
	-class 类加载信息统计
	-compiler 及时编译信息统计
	-gc 垃圾收集堆信息统计
	-gccapacity 虚拟机各代内存容量和当前空间信息统计
	-gccause 垃圾收集信息统计，同gcutil，并给出最后一次和当前正在执行的GC事件
	-gcnew 新生代空间及GC信息统计
	-gcnewcapacity 新生代容量及当前空间信息统计
	-gcold 年老代和持久带空间及GC信息统计
	-gcoldcapacity 年老代信息统计
	-gcpermcapacity 持久带信息统计
	-gcutil 常用的GC信息统计
	-printcompilation 编译信息打印


gcutil输出结果说明：
	
	[gmz@server01 ~]$ jstat -gcutil 18077 2s 3
	S0 S1 E O P YGC YGCT FGC FGCT GCT
	0.00 0.00 16.34 1.36 56.38 3 0.054 2 0.520 0.574
	0.00 0.00 16.34 1.36 56.38 3 0.054 2 0.520 0.574
	0.00 0.00 16.34 1.36 56.38 3 0.054 2 0.520 0.574

+ E 新生代（New Generation）中Eden Space，用于存放新创建的对象，显示使用率
+ S0 新生代中的幸存代0（即From Space），用于存放E中幸存下来的对象，显示使用率
+ S1 新生代中的幸存代1（即To Space），用于存放S0中幸存下来的对象，显示使用率
+ O 年老代（Old Space），用于存放S1中幸存下来的对象，显示使用率
+ P 持久带（Permanent Space），用于存放用于存放对象的Class实例和反射代理，显示使用率。类加载过程包括装载、链接和初始化，在装载时首先根据类的全限定名找到类的二进制数据，解析后存到方法区中，完了就创建类的Class实例，存放到持久带。
+ YGC 新生代GC次数
+ YGCT 新生代GC时间
+ FGC FULL GC次数，对新生代和年老代进行的GC
+ FGCT Full GC时间
+ GCT YGCT+FGCT

其他统计项输出结果可参见原文[jstat.html](http://java.sun.com/javase/6/docs/technotes/tools/share/jstat.html)

## 3. jstack 追踪JVM线程栈信息

+ jstack [ option ] pid
+ jstack [ option ] executable core
+ jstack [ option ] [server-id@]remote-hostname-or-IP

示例：
	
	[gmz@server01 ~]$ jstack 18077
	2011-07-21 17:17:24
	Full thread dump Java HotSpot(TM) 64-Bit Server VM (20.0-b11 mixed mode):
	
	“Attach Listener” daemon prio=10 tid=0x00000000468c7800 nid=0x4c6d waiting on condition [0x0000000000000000]
	java.lang.Thread.State: RUNNABLE
	
	“GC Daemon” daemon prio=10 tid=0x0000000046de0000 nid=0x46d8 in Object.wait() [0x0000000042039000]
	java.lang.Thread.State: TIMED_WAITING (on object monitor)
	at java.lang.Object.wait(Native Method)
	– waiting on <0x0000000783027fe8> (a sun.misc.GC$LatencyLock)
	at sun.misc.GC$Daemon.run(GC.java:100)
	– locked <0x0000000783027fe8> (a sun.misc.GC$LatencyLock)

选项：

+ -l 附加锁信息
+ -F 强制输出栈信息
+ -m 同时输出java栈和本地方法栈信息
+ -h 或 -help 输出帮助信息

其他还有jstatd jmap jhat jinfo jvisualvm jconsole等，具体可参考：[JVM Tools](http://download.oracle.com/javase/6/docs/technotes/tools/index.html) 



## references
+ [JVM Tools](http://docs.oracle.com/javase/6/docs/technotes/tools/index.html)
