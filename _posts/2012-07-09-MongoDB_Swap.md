---
layout: post
title: MongoDB swap 释疑
category : MongoDB
tags : [MongoDB]
---
{% include JB/setup %}

`MongoDB Swap`

## 1。 issue: 可疑的swap占用
Swap，即交换区，当系统的物理内存不够用的时候，就需要将物理内存中的一部分空间释放出来，以供当前运行的程序使用。
那些被释放的空间可能来自一些很长时间没有什么操作的程序，这些被释放的空间被临时保存到Swap空间中，等到那些程序要运行时，再从Swap中恢复保存的数据到内存中。  
线上服务最近发现`swap used`经常默默地就涨到了1G多，很少能够及时回收甚至一直used不变。这是为何？

## 2。 swap使用情况查看
**swapon**  
使用swapon可以查看swap使用情况，如下

	[root@mg1sh gmz]# /sbin/swapon -s
	Filename                                Type            Size    Used    Priority
	/dev/sda5                               partition       12289684        1699092 -1

swapon -s，显示的是swap交换区使用概况  

**top**  
通过top命令可以查看swap使用情况，如下

	[mba@mg1sh ~]$ top -umba
	top - 14:26:20 up 193 days, 7 min,  2 users,  load average: 0.66, 0.58, 0.50
	Tasks: 170 total,   1 running, 169 sleeping,   0 stopped,   0 zombie
	Cpu(s):  0.0%us,  0.0%sy,  0.0%ni, 99.9%id,  0.0%wa,  0.0%hi,  0.0%si,  0.0%st
	Mem:  49448636k total, 49276716k used,   171920k free,   130216k buffers
	Swap: 12289684k total,  1699092k used, 10590592k free, 45243008k cached
	  PID USER      PR  NI  VIRT  RES  SHR S %CPU %MEM    TIME+  SWAP COMMAND                                                                                               
	 3115 mba       15   0  109g  25g  24g S 11.0 53.2  43430:54  84g mongod                                                                                                
	 3506 mba       15   0  631m  28m  14m S  0.3  0.1 553:30.67 603m mongod                                                                                                
	20387 mba       15   0 12736 1144  820 R  0.3  0.0   0:00.13  11m top                                                                                                   
	 3371 mba       15   0 89.0g 4.4g 4.3g S  0.0  9.4 295:47.42  84g mongod                                                                                                
	 4401 mba       15   0 76.8g 5.0g 4.9g S  0.0 10.6 157:27.81  71g mongod                                                                                                
	 6176 mba       15   0  628m 157m 2728 S  0.0  0.3 277:14.62 471m mongos                                                                                                
	12561 mba       18   0 4541m 1.6g 5684 S  0.0  3.4   1:20.25 2.8g java                                                                                                  
	12562 mba       18   0  3836  504  404 S  0.0  0.0   0:00.72 3332 cronolog                                                                                              
	20037 mba       16   0 66100 1600 1176 S  0.0  0.0   0:00.02  62m bash                                                                                                  
	
top查到swap总使用量约1659M，与swapon结果一致。  
top回车后，再f键、p键可以查看每个进程的swap，这里的swap含义并非当前占用的swap大小，而是可以使用的swap最大值。  
shift+m，可按RES排序。  

**free**  
使用free命令查看swap如下

	[root@mg1sh gmz]# free -k
	             total       used       free     shared    buffers     cached
	Mem:      49448636   49287456     161180          0     133072   45250648
	-/+ buffers/cache:    3903736   45544900
	Swap:     12289684    1699092   10590592

free查到swap总使用量1659M，与swapon、top查到的一样。    

**smaps**  
参照[Find out what is using your swap](http://northernmost.org/blog/find-out-what-is-using-your-swap/)一文，
从/proc/进程ID/smaps中可以统计出某个进程的swap使用情况，整理后的脚本如下

	#!/bin/bash
	# Get current swap usage for all running processes
	# Erik Ljungstrom 27/05/2011
	# updated by gmz, 20120709	
	total=0	
	users=(`cat /etc/passwd |awk 'BEGIN{FS=":";}{print $1;}'`)
	#echo ${users[*]}
	#exit
	for user in ${users[*]}; do
	        sum=0;
	        procs=(`ps -u$user -o pid --no-headers`)
	        for pid in ${procs[*]}; do
	                pn=`ps -p $pid -o cmd --no-headers`
	                swap=`cat /proc/$pid/smaps |grep Swap|awk 'BEGIN{sum=0;}{sum=sum+$2;}END{print sum;}'`
	                if [ $swap -gt 0 ]; then
	                        echo "$pid      ${swap}k        $pn"
	                        let sum=$sum+$swap
	                fi
	        done
	        if [ $sum -gt 0 ];then
	                echo "$user used swap: ${sum}k"
	                echo ""
	                let total=$total+$sum
	        fi
	done
	echo ""
	echo "total used swap: ${total}k"	
	exit

改脚本需要root账号才能执行，结果如下

	[root@mg1sh gmz]# sh /home/mba/myswap.sh 
	……
	root used swap: 357652k	
	……
	haldaemon used swap: 2164k	
	……
	zabbix used swap: 1128k	
	……
	mba used swap: 573900k		
	total used swap: 935220k

得到swap总使用量与前面的结果不一样，且相差较大，怀疑该方法的准确性，具体区别待确认。  


## 3. 问题定位
从上面top命令结果可见，除了mongo进程还有一个java进程，为免除意外干扰，重启java进程，再top结果如下

	Production: [mba@mg1sh] ~$ top -umba
	top - 16:23:53 up 194 days,  2:04,  1 user,  load average: 0.10, 0.03, 0.04
	Tasks: 166 total,   2 running, 164 sleeping,   0 stopped,   0 zombie
	Cpu(s):  0.0%us,  0.7%sy,  2.7%ni, 96.6%id,  0.0%wa,  0.0%hi,  0.0%si,  0.0%st
	Mem:  49448636k total, 49269456k used,   179180k free,   160888k buffers
	Swap: 12289684k total,   973892k used, 11315792k free, 46651800k cached
	
	  PID USER      PR  NI  VIRT  RES  SHR S %CPU %MEM    TIME+  SWAP COMMAND                                                                                               
	 3115 mba       15   0  113g  24g  24g S  0.0 52.5  44078:34  88g mongod                                                                                                
	 4401 mba       15   0 76.8g 4.2g 4.0g S  0.0  8.8 165:50.17  72g mongod                                                                                                
	 3371 mba       15   0 93.0g 3.4g 3.3g S  0.0  7.2 304:40.36  89g mongod                                                                                                
	26263 mba       18   0 4464m 274m  10m S  0.0  0.6   0:10.32 4.1g java                                                                                                  
	 6176 mba       15   0  637m 163m 2808 S  0.0  0.3 285:22.30 474m mongos                                                                                                
	 3506 mba       15   0  638m  45m  31m S  0.8  0.1 562:44.56 592m mongod                                                                                                
	  405 mba       16   0 66228 1660 1208 S  0.0  0.0   0:00.27  63m bash                                                                                                  
	 9024 mba       15   0 12736 1136  820 R  0.8  0.0   0:00.02  11m top                                                                                                   
	26264 mba       18   0  3836  480  404 S  0.0  0.0   0:00.04 3356 cronolog    

可见swap有些回升，used降到900M左右。  
继续查阅[Checking Server Memory Usage](http://www.mongodb.org/display/DOCS/Checking+Server+Memory+Usage)、[Caching](http://www.mongodb.org/display/DOCS/Caching)
两篇官方doc，怀疑和客户端连接有关。mongod/mongos会为每个客户端连接创建一个线程，每个线程会有一个stack，2.0版本后默认栈大小应该是`the lesser of the system setting or 1MB`，实际验证下：

	Production: [root@mg1sh] /home/mba$ ulimit -a|grep stack
	stack size              (kbytes, -s) 10240
	
	Production: [root@mg1sh] /home/mba$ cat /proc/3115/limits |grep stack
	Max stack size            10485760             unlimited            bytes
	
	Production: [root@mg1sh] /home/mba$ cat /proc/3115/smaps | grep 10240 -A 6 -B 1 
	40a22000-41422000 rw-p 40a22000 00:00 0 
	Size:             10240 kB
	Rss:                  0 kB
	Shared_Clean:         0 kB
	Shared_Dirty:         0 kB
	Private_Clean:        0 kB
	Private_Dirty:        0 kB
	Swap:        8 kB
	--  
	……

可见实际使用的stack大小为10M，why？待确定。。。  
可以使用`ulimit -s 1024`更改系统默认stack大小，但需要重启mongod，先以MongoDB 27028（PID3371）进程进行测试，执行记录如下：

	// 先看下swap used有973892K
	Production: [root@mg1sh] /home/mba$ free
	             total       used       free     shared    buffers     cached
	Mem:      49448636   49277572     171064          0     165760   46653160
	-/+ buffers/cache:    2458652   46989984
	Swap:     12289684     973892   11315792
	
	// top看到进程27028（PID3371）RES为3.4g，paging过的SWAP为89g
	Production: [root@mg1sh] /home/mba$ top -umba
	top - 17:06:48 up 194 days,  2:47,  1 user,  load average: 0.16, 0.06, 0.01
	Tasks: 165 total,   1 running, 164 sleeping,   0 stopped,   0 zombie
	Cpu(s):  0.0%us,  0.1%sy,  0.0%ni, 99.9%id,  0.0%wa,  0.0%hi,  0.0%si,  0.0%st
	Mem:  49448636k total, 49277780k used,   170856k free,   165796k buffers
	Swap: 12289684k total,   973892k used, 11315792k free, 46653160k cached
	
	  PID USER      PR  NI  VIRT  RES  SHR S %CPU %MEM    TIME+  SWAP COMMAND                                                                                               
	 3115 mba       15   0  113g  24g  24g S  0.0 52.5  44078:37  88g mongod                                                                                                
	 4401 mba       15   0 76.8g 4.2g 4.0g S  0.0  8.8 165:50.82  72g mongod                                                                                                
	 3371 mba       15   0 93.0g 3.4g 3.3g S  0.0  7.2 304:40.98  89g mongod                                                                                                
	26263 mba       18   0 4482m 291m  10m S  0.0  0.6   0:11.81 4.1g java                                                                                                  
	 6176 mba       15   0  637m 163m 2808 S  0.0  0.3 285:24.38 474m mongos                                                                                                
	 3506 mba       15   0  638m  45m  31m S  0.0  0.1 563:05.30 592m mongod                                                                                                
	26264 mba       18   0  3836  480  404 S  0.0  0.0   0:00.04 3356 cronolog                                                                                              
	
	// 27028（PID3371） 为SECONDARY，副本集中的二级节点，因此重启不会影响线上正常服务
	Production: [root@mg1sh] /home/mba$ ./mongodb-2.0.4/bin/mongo localhost:27028
	MongoDB shell version: 2.0.4
	connecting to: localhost:27028/test
	SECONDARY> exit
	bye
	
	// 退回MongoDB账号mba
	Production: [root@mg1sh] /home/mba/mongodb-2.0.4$ exit
	exit
	Production: [gmz@mg1sh] ~$ sudo su mba
	Production: [mba@mg1sh] /home/gmz$ cd
	Production: [mba@mg1sh] ~$
	
	// 临时修改默认stack size
	Production: [mba@mg1sh] ~$ ulimit -a|grep stack
	stack size              (kbytes, -s) 10240
	Production: [mba@mg1sh] ~$ ulimit -s 1024
	Production: [mba@mg1sh] ~$ ulimit -a|grep stack
	stack size              (kbytes, -s) 1024
	
	// 重启27028
	Production: [mba@mg1sh] ~$ kill 3371
	Production: [mba@mg1sh] ~$ ./mongodb-2.0.4/bin/mongod --port 27028 --replSet shard02/mg2sh.prod.mediav.com:27028 --fork --dbpath /data/mba-data/shard02/ --oplogSize 20480 --logpath /data1/mba-logs/shard02/mongod.log --logappend
	all output going to: /data1/mba-logs/shard02/mongod.log
	forked process: 15520

	// 验证新的27028（PID15520）所用栈大小，确为1024K
	Production: [mba@mg1sh] ~$ cat /proc/15520/limits |grep stack
	Max stack size            1048576              1048576              bytes     
	Production: [mba@mg1sh] ~$ 	
	Production: [mba@mg1sh] ~$ cat /proc/15520/smaps |grep 1024 -B 2 -A 6 |head -10
	Swap:        0 kB
	40301000-40401000 rw-p 40301000 00:00 0 
	Size:              1024 kB
	Rss:                  8 kB
	Shared_Clean:         0 kB
	Shared_Dirty:         0 kB
	Private_Clean:        0 kB
	Private_Dirty:        8 kB
	Swap:        0 kB
	--
	
	// 重启后释放了约100M的swap
	Production: [mba@mg1sh] ~$ free
	             total       used       free     shared    buffers     cached
	Mem:      49448636   49272492     176144          0     169488   46682492
	-/+ buffers/cache:    2420512   47028124
	Swap:     12289684     815992   11473692
	Production: [mba@mg1sh] ~$ top -umba
	top - 17:34:58 up 194 days,  3:15,  1 user,  load average: 0.00, 0.00, 0.00
	Tasks: 165 total,   1 running, 164 sleeping,   0 stopped,   0 zombie
	Cpu(s):  0.0%us,  0.3%sy,  0.0%ni, 99.7%id,  0.0%wa,  0.0%hi,  0.0%si,  0.0%st
	Mem:  49448636k total, 49272208k used,   176428k free,   169516k buffers
	Swap: 12289684k total,   815992k used, 11473692k free, 46682492k cached
	
	  PID USER      PR  NI  VIRT  RES  SHR S %CPU %MEM    TIME+  SWAP COMMAND                                                                                               
	 3115 mba       15   0  113g  24g  24g S  0.0 52.7  44078:40  88g mongod                                                                                                
	 4401 mba       15   0 76.8g 4.2g 4.0g S  0.0  8.8 165:51.19  72g mongod                                                                                                
	26263 mba       18   0 4603m 460m  10m S  0.0  1.0   0:18.87 4.0g java                                                                                                  
	 6176 mba       15   0  637m 163m 2864 S  0.0  0.3 285:26.05 474m mongos                                                                                                
	 3506 mba       15   0  638m  45m  31m S  0.0  0.1 563:18.92 592m mongod                                                                                                
	15520 mba       15   0 40.5g  30m  19m S  0.0  0.1   0:00.09  40g mongod                                                                                                
	15339 mba       16   0 66232 1652 1204 S  0.0  0.0   0:00.10  63m bash                                                                                                  
	15915 mba       15   0 12736 1140  820 R  0.0  0.0   0:00.02  11m top                                                                                                   
	26264 mba       18   0  3836  480  404 S  0.0  0.0   0:00.05 3356 cronolog                                                                                              

	// 27028 有12个连接
	Production: [mba@mg1sh] ~$ ./mongodb-2.0.4/bin/mongostat -h localhost --port 27028
	connected to: localhost:27028
	insert  query update delete getmore command flushes mapped  vsize    res faults locked % idx miss %     qr|qw   ar|aw  netIn netOut  conn     set repl       time 
	    *0     *0     *0     *0       0     1|0       0  20.2g  40.5g    31m      0        0          0       0|0     0|0    62b     1k    12 shard02  SEC   17:43:39 

如果所猜没错，12个连接原先共占用120M内存，重启后只占用12M内存，如果这些连接都空闲并被swap了，所以交换区能够回收约100M。perfect！！！  
但是这种方式是否能够真正fix swap诡异增长的问题，还待继续观察。  

=================happy split line=================   
OK，猜测是错的。  
MongoDB2.0以上default stack size确实是取得`the lesser of the system setting or 1MB`，只是这个是在thread创建时确定的。
参见[default stack size](https://groups.google.com/forum/?fromgroups#!topic/mongodb-user/GOAOwYH483c)里Eliot最后回复我的追问。  
另外还有一问，参见[will the server closes the client connection automatically](https://groups.google.com/forum/?fromgroups#!topic/mongodb-user/IdSp7WcA66o)，
mongod/mongos不会自动关闭客户端连接，即便连接已经超时，这个工作是由OS/TCP完成的，当然会有些延迟。如果sockettimeout指定30min，实际操作几秒钟就完成了，为了保险client最好主动关闭连接。  
<br>

**the real answer**  
1. 查询MongoDB日志发现有垃圾数据造成balancer失败，日志如下

	Wed Jul 11 03:18:02 [Balancer] balancer move failed: { chunkTooBig: true, estimatedChunkSize: 163939008, errmsg: "chunk too big to move", ok: 0.0 } from: shard01 to: shard03 chunk: { _id: "mba.user.base-sid_"m-20212-1"stt_20120401edt_20120430", lastmod: Timestamp 23000|1, ns: "mba.user.base", min: { sid: "m-20212-1", stt: 20120401, edt: 20120430 }, max: { sid: "m-20212-1", stt: 20120402, edt: 20120402 }, shard: "shard01" }

删除垃圾数据，balancer成功。  
2. 修改程序代码，及时关闭客户端连接。  
3. 此外，重启了集群中部分mongod进程（configdb和replSet中的primary节点没有重启），使得swap恢复到合理范围。  
经过这三项调整后，观察一周swap一直正常。


## references
+ [Linux Swap之谜](http://unix-cd.com/vc/www/28/2009-02/13513.html)
+ [Find out what is using your swap](http://northernmost.org/blog/find-out-what-is-using-your-swap/)
+ [Linux系统中Java程序占用swap空间的问题](http://www.iteye.com/topic/162526)
+ [释放swap虚拟内存的方法](http://www.unix-center.net/bbs/viewthread.php?tid=13006)
+ [MySQL如何避免使用Linux的swap分区](http://tieba.baidu.com/p/1210406666?pid=14267091463&cid=0)
+ [Checking Server Memory Usage](http://www.mongodb.org/display/DOCS/Checking+Server+Memory+Usage)
+ [Caching](http://www.mongodb.org/display/DOCS/Caching)
+ [MongoDB与内存](http://huoding.com/2011/08/19/107)
+ [MongoDB性能优化之连接优化](http://blog.csdn.net/dragon8299/article/details/6776142)
+ [important config note on running MongoDB in production on Linux](https://groups.google.com/forum/?fromgroups#!topic/mongodb-user/GOAOwYH483c)
+ [2.0 Release Notes](http://www.mongodb.org/display/DOCS/2.0+Release+Notes)
+ [Linux Programmer's Manual](http://www.kernel.org/doc/man-pages/online/pages/man5/proc.5.html)
+ [Linux Programmer's Manual](http://www.kernel.org/doc/man-pages/online/pages/man5/proc.5.html)
+ [Linux Programmer's Manual](http://www.kernel.org/doc/man-pages/online/pages/man5/proc.5.html)
