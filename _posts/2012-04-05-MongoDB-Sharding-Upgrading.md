---
layout: post
title: MongoDB集群环境升级
category : MongoDB
tags : [MongoDB, Sharding, replSet]
---
{% include JB/setup %}

`MongoDb Sharding Upgrading`

在一台测试机上搭建了MongoDB的sharding环境，对sharding环境的upgrading进行了简单测试，记录如下
## 原Sharding架构
原先只有一个configdb，两个分片都是副本集，每个分片两个节点，具体如下

	configdb			we1bo:27019
	shard01/replSet		we1bo:27018(primary),we1bo:27028
	shard02/replSet		we1bo:27068(primary),we1bo:27078
	mongos				we1bo:27017


## 新增副本
添加新副本比较简单，先启动新的mongod节点，然后使用rs.add把新节点添加到副本集中即可。具体操作如下  

+ shard01当前配置
	通过rs.config()可以查看当前副本集配置

	[mba@server01 ~]$ cd /home/mba/local/mongodb-2.0.2
	[mba@server01 ~]$ ./bin/mongo we1bo.corp.mediav.com:27018
	MongoDB shell version: 2.0.2
	connecting to: we1bo.corp.mediav.com:27018/test
	PRIMARY> 
	PRIMARY> rs.config()
	{
	        "_id" : "shard01",
	        "version" : 2,
	        "members" : [
	                {
	                        "_id" : 1,
	                        "host" : "we1bo.corp.mediav.com:27018",
	                        "priority" : 2
	                },
	                {
	                        "_id" : 2,
	                        "host" : "we1bo.corp.mediav.com:27028"
	                }
	        ]
	}


+ 启动新节点
	直接启动一个新的mongod实例即可

	./bin/mongod --port 27038 --replSet shard01/we1bo.corp.mediav.com:27018 --fork --dbpath /home/mba/data/data/shard01/node03 --oplogSize 1024 --logpath /home/mba/data/logs/shard01/node03/mongod.log


+ 添加新节点
	连接到shard01首节点，使用rs.add直接添加即可

	./bin/mongo we1bo.corp.mediav.com:27018/admin
	rs.add({_id: 3, host: "we1bo.corp.mediav.com:27038", priority: 1})


## 新增configdb
本以为只需要暂停configdb、mongos进程，然后在启动新的configdb实例，再使用新的configdb参数启动mongos即可，结果测试失败，总是报错

	Thu Apr  5 14:52:33 [Balancer] caught exception while doing balance: could not initialize sharding on connection we1bo.corp.mediav.com:27019 :: caused by :: specified a different configdb!


必须把所有mongod、configdb、mongos都暂停掉，其实configdb也就是一般的mongod进程

+ 暂停所有mongo进程
	先kill mongos进程，第一次kill的时候半天没有反应，以为是有client连接造成，关闭所有client连接后仍然没反应，考虑到当前没有对MongoDB的数据更新操作，直接kill -9 1879了，最终kill掉了，也没有造成数据错误。
后来测试在有client连接时使用无参kill也可正常杀掉mongos进程，想到一开始执行的添加副本操作，可能因此使得第一次kill失败。
下面再依次kill掉configdb、secondary副本、primary副本。

	[mba@server01 mongodb-2.0.2]$ ps aux|grep mongo
	mba       1879  0.0  0.0 296724  3456 ?        Sl   16:00   0:00 ./bin/mongos --port 27017 --configdb localhost:27019 --fork --logpath /home/mba/data/logs/mongos.log --logappend
	mba      32457  0.0  0.1 454616 33284 ?        Sl   15:33   0:00 ./bin/mongod --port 27019 --fork --dbpath /home/mba/data/data/configdb --oplogSize 1024 --logpath /home/mba/data/logs/configdb/mongod.log
	mba      32528  0.0  0.1 118051564 62772 ?     Sl   15:34   0:00 ./bin/mongod --port 27018 --replSet shard01/we1bo.corp.mediav.com:27028 --fork --dbpath /home/mba/data/data/shard01/node01 --oplogSize 1024 --logpath /home/mba/data/logs/shard01/node01/mongod.log
	mba      32560  0.0  0.0 4980792 31740 ?       Sl   15:34   0:00 ./bin/mongod --port 27028 --replSet shard01/we1bo.corp.mediav.com:27018 --fork --dbpath /home/mba/data/data/shard01/node02 --oplogSize 1024 --logpath /home/mba/data/logs/shard01/node02/mongod.log
	mba      32607  0.0  0.0 4883640 31572 ?       Sl   15:34   0:00 ./bin/mongod --port 27038 --replSet shard01/we1bo.corp.mediav.com:27018 --fork --dbpath /home/mba/data/data/shard01/node03 --oplogSize 1024 --logpath /home/mba/data/logs/shard01/node03/mongod.log
	mba      32642  0.0  0.0 4859932 31528 ?       Sl   15:34   0:00 ./bin/mongod --port 27068 --replSet shard02/we1bo.corp.mediav.com:27078 --fork --dbpath /home/mba/data/data/shard02/node01 --oplogSize 1024 --logpath /home/mba/data/logs/shard02/node01/mongod.log
	mba      32670  0.0  0.0 4933728 31884 ?       Sl   15:34   0:00 ./bin/mongod --port 27078 --replSet shard02/we1bo.corp.mediav.com:27068 --fork --dbpath /home/mba/data/data/shard02/node02 --oplogSize 1024 --logpath /home/mba/data/logs/shard02/node02/mongod.log
	
	[mba@server01 mongodb-2.0.2]$ kill 1879
	[mba@server01 mongodb-2.0.2]$ kill 32457
	[mba@server01 mongodb-2.0.2]$ kill 32607
	[mba@server01 mongodb-2.0.2]$ kill 32560
	[mba@server01 mongodb-2.0.2]$ kill 32528
	[mba@server01 mongodb-2.0.2]$ kill 32670
	[mba@server01 mongodb-2.0.2]$ kill 32642


+ 启动新的configdb实例
	configdb要不是1台，要不是3台或更多，一般3台即可满足灾备需求，所以新增2台即可。
直接复制原configdb的data目录

	[mba@server01 ~]$ cd /home/mba/data/data/
	[mba@server01 data]$ cp -r configdb/ configdb2
	[mba@server01 data]$ cp -r configdb/ configdb3

再启动新的configdb

	[mba@server01 ~]$ cd /home/mba/local/mongodb-2.0.2
	[mba@server01 mongodb-2.0.2]$ ./bin/mongod --port 27019 --fork --dbpath /home/mba/data/data/configdb --oplogSize 1024 --logpath /home/mba/data/logs/configdb/mongod.log
	[mba@server01 mongodb-2.0.2]$ ./bin/mongod --port 27029 --fork --dbpath /home/mba/data/data/configdb2 --oplogSize 1024 --logpath /home/mba/data/logs/configdb2/mongod.log
	[mba@server01 mongodb-2.0.2]$ ./bin/mongod --port 27039 --fork --dbpath /home/mba/data/data/configdb3 --oplogSize 1024 --logpath /home/mba/data/logs/configdb3/mongod.log


+ 重启所有mongo进程
	按官方文档[Upgrading from one config server to three](http://www.mongodb.org/display/DOCS/Changing+Config+Servers#ChangingConfigServers-Upgradingfromoneconfigservertothree)
先重启mongos进程再重启mongod进程会出错，错误如下

	Thu Apr  5 15:34:57 [Balancer] warning: could not initialize balancer, please check that all shards and config servers are up: socket exception
	Thu Apr  5 15:34:57 [Balancer] will retry to initialize balancer in one minute
	Thu Apr  5 15:35:06 [mongosMain] connection accepted from 127.0.0.1:60501 #1
	Thu Apr  5 15:35:15 [conn1] DBException in process: socket exception

应该先重启所有mongod、configdb进程，再启动mongos。configdb已经启动，此处只要启动所有mongod即可，注意先启动primary节点，再启动secondary节点，以防有什么意外发生。

	[mba@server01 mongodb-2.0.2]$ ./bin/mongod --port 27018 --replSet shard01/we1bo.corp.mediav.com:27028 --fork --dbpath /home/mba/data/data/shard01/node01 --oplogSize 1024 --logpath /home/mba/data/logs/shard01/node01/mongod.log
	[mba@server01 mongodb-2.0.2]$ ./bin/mongod --port 27028 --replSet shard01/we1bo.corp.mediav.com:27018 --fork --dbpath /home/mba/data/data/shard01/node02 --oplogSize 1024 --logpath /home/mba/data/logs/shard01/node02/mongod.log
	[mba@server01 mongodb-2.0.2]$ ./bin/mongod --port 27038 --replSet shard01/we1bo.corp.mediav.com:27018 --fork --dbpath /home/mba/data/data/shard01/node03 --oplogSize 1024 --logpath /home/mba/data/logs/shard01/node03/mongod.log
	[mba@server01 mongodb-2.0.2]$ ./bin/mongod --port 27068 --replSet shard02/we1bo.corp.mediav.com:27078 --fork --dbpath /home/mba/data/data/shard02/node01 --oplogSize 1024 --logpath /home/mba/data/logs/shard02/node01/mongod.log
	[mba@server01 mongodb-2.0.2]$ ./bin/mongod --port 27078 --replSet shard02/we1bo.corp.mediav.com:27068 --fork --dbpath /home/mba/data/data/shard02/node02 --oplogSize 1024 --logpath /home/mba/data/logs/shard02/node02/mongod.log

最后使用新的configdb参数启动mongos

	[mba@server01 mongodb-2.0.2]$ ./bin/mongos --port 27017 --configdb localhost:27019,localhost:27029,localhost:27039 --fork --logpath /home/mba/data/logs/mongos.log --logappend

最后连接mongos 27017端口，查看分片状态，可以看到MongoDB集群已经正常起来了

	[mba@server01 ~]$ ./local/mongodb-2.0.2/bin/mongo we1bo.corp.mediav.com:27017
	MongoDB shell version: 2.0.2
	connecting to: we1bo.corp.mediav.com:27017/test
	mongos> db.printShardingStatus()
	--- Sharding Status --- 
	  sharding version: { "_id" : 1, "version" : 3 }
	  shards:
	        {  "_id" : "shard01",  "host" : "shard01/we1bo.corp.mediav.com:27018,we1bo.corp.mediav.com:27038,we1bo.corp.mediav.com:27028" }
	        {  "_id" : "shard02",  "host" : "shard02/we1bo.corp.mediav.com:27068,we1bo.corp.mediav.com:27078" }
	  databases:
	        ……


### references
+ [Replica Sets](http://www.mongodb.org/display/DOCS/Replica+Sets)
+ [Changing Config Servers](http://www.mongodb.org/display/DOCS/Changing+Config+Servers)

