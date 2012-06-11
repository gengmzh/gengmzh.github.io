---
layout: post
title: MongoDB丢失数据
category : MongoDB
tags : [MongoDB]
---
{% include JB/setup %}

`MongoDB Missing Data`  


## issue
项目中用到MongoDB，搭建的一个3x3的集群（三个分片，每个分片三个副本），导库任务每天执行一次，导入各站点的统计数据到MongoDB中。
最近发现有数据丢失情况，但导库日志中并没有报错，重跑后数据又正常了。

## 问题定位
**WriteConcern**  
MongoDB默认是异步写，`WriteConcern.NORMAL`，只会报一些网络异常，不管mongod有没有真的写入数据。因此在导库日志中看不到数据丢失相关的错误。
问题定位的第一步就是改用安全模式，`WriteConcern.SAFE`，不但会报网络错误，还会执行getLastError，等待mongod返回insert结果。


## references
+ [getLastError](http://www.mongodb.org/display/DOCS/getLastError+Command)
+ [Java Driver Concurrency](http://www.mongodb.org/display/DOCS/Java+Driver+Concurrency)

