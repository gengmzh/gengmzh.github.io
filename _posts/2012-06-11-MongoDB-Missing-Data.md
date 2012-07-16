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

## fix
**WriteConcern**  
MongoDB默认是异步写，`WriteConcern.NORMAL`，只会报一些网络异常，不管mongod有没有真的写入数据。因此在导库日志中看不到数据丢失相关的错误。
问题定位的第一步就是改用安全模式，`WriteConcern.SAFE`，不但会报网络错误，还会执行getLastError，等待mongod返回insert结果。

---
happy split line  

修改为SAFE模式，观察两周没有再发先有数据丢失情况。

## CursorNotFound: cursor not found on server
随着数据量增加MongoDB问题也不断涌现啊  
*Whenever you issue a query, a cursor is created on the server. Cursor naturally time out after ten minutes, which means that if you happen to be iterating over a cursor for more than ten minutes, you risk a CURSORNOTFOUND exception.*  
从官方文档上找到的明确的解释，查看日志如下

	12/06/27 04:29:17 INFO uservalue.Loader: site m-189-0: merge by Week done
	12/06/27 04:29:17 INFO uservalue.Loader: site m-189-0: merge by Month begin
	12/06/27 04:50:29 FATAL uservalue.Loader: site m-189-0 failed
	12/06/27 04:50:29 FATAL uservalue.Loader: load site m-189-0 failed

可见从上一条INFO日志到第一条FATAL日志，确实超出了10分钟，为何这里会停顿这么久？？？  
怀疑是并发访问造成，尝试调整并发数及MongoOptions参数，并发数调为3，参数调为connecttimeoutms=300000;sockettimeoutms=1800000;，观察一周问题没有再出现。


## references
+ [getLastError](http://www.mongodb.org/display/DOCS/getLastError+Command)
+ [Java Driver Concurrency](http://www.mongodb.org/display/DOCS/Java+Driver+Concurrency)

