---
layout: post
title: Inside Hadoop, Metrics2 System
category : hadoop
tags : [hadoop, metrics2]
---
{% include JB/setup %}


`Metrics2 System of Hadoop 1.0.3`  
  
metrics/metrics2，是Hadoop监控系统的基本框架，通过它可以监控到DataNode、NameNode、JobTracker、TaskTracker等各个组件的运行情况，并且原生支持ganglia非常方便。  

### 1. metrics使用
生产环境使用的是`hadoop-0.20.2-cdh3u4`，cloudera发布的企业版，配置起来更加简单，直接在配置文件里打开对应的开关即可，如下：  

	cat conf/hadoop-metrics.properties 
	# Configuration of the "dfs" context for null
	# dfs.class=org.apache.hadoop.metrics.spi.NullContext
	
	# Configuration of the "dfs" context for file
	dfs.class=org.apache.hadoop.metrics.file.FileContext
	dfs.period=10
	dfs.fileName=/tmp/dfsmetrics.log
	
	# Configuration of the "dfs" context for ganglia
	# Pick one: Ganglia 3.0 (former) or Ganglia 3.1 (latter)
	# dfs.class=org.apache.hadoop.metrics.ganglia.GangliaContext
	# dfs.class=org.apache.hadoop.metrics.ganglia.GangliaContext31
	# dfs.period=10
	# dfs.servers=localhost:8649

开启了dfs监控，监控结果直接输出到文件，重启datanode后可以看到metrics已经有输出了：

	head -1 /tmp/dfsmetrics.log      
	dfs.datanode: hostName=web109, sessionId=, blockChecksumOp_avg_time=0, blockChecksumOp_num_ops=0, blockReports_avg_time=555, blockReports_num_ops=1, block_verification_failures=0, blocks_read=46, blocks_removed=37, blocks_replicated=0, blocks_verified=0, blocks_written=0, bytes_read=3064524, bytes_written=0, copyBlockOp_avg_time=0, copyBlockOp_num_ops=0, heartBeats_avg_time=3, heartBeats_num_ops=2, readBlockOp_avg_time=6, readBlockOp_num_ops=46, reads_from_local_client=46, reads_from_remote_client=0, replaceBlockOp_avg_time=0, replaceBlockOp_num_ops=0, volumeFailures=0, writeBlockOp_avg_time=0, writeBlockOp_num_ops=0, writes_from_local_client=0, writes_from_remote_client=0

从中可以看到：

+ readBlockOp_num_ops=46：本次监控间隔里读取了46个文件块
+ readBlockOp_avg_time=6：平均每次读取耗时6毫秒
+ writeBlockOp_num_ops=0：本次监控间隔里没有写文件操作
+ writeBlockOp_avg_time=0：没有写操作，因此每次写耗时0毫秒


### 2. Metrics2 Architecture
最新的metrics2设计非常清晰，类似于生产者消费者模型，MetricsSource是生产者负责收集系统运行数据，MetricsSink是消费者负责将source收集到的数据发布出去。总体结构如下所示：  
![Metrics2 System Overview](https://issues.apache.org/jira/secure/attachment/12452869/metrics2-uml-r2.png)  

这个监控系统主要有三个组成部分：  

+ MetricsSystem：总控制类，负责读取配置信息初始化MetricsSource、MetricsSink和自身，对MetricsSource和MetricsSink进行进一步封装和适配，并在主进程中启动监控定时器，定期从MetricsSource读取监控信息发给MetricsSink。
+ MetricsSource：监控信息源，负责收集监控信息，由监控定时器定时调用，本身不起用独立线程。
+ MetricsSink：监控信息发布者，警MetricsSystem进一步封装后的适配类中会为每一个MetricsSink启动一个线程，定期从监控信息队列中获取监控信息。


### 3. 使用Zabbix监控hadoop运行状况
TODO

### 4. 使用metrics2框架监控hadoop job
TODO

### references
+ [Metrics2 Package Summary](http://hadoop.apache.org/docs/r1.0.3/api/org/apache/hadoop/metrics2/package-summary.html)
+ [Metrics2 Design Document](http://wiki.apache.org/hadoop/HADOOP-6728-MetricsV2)
+ [Hadoop-CDH Metrics Configuration](http://blog.cloudera.com/blog/2009/03/hadoop-metrics/)

