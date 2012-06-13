---
layout: post
title: MongoDB索引优化
category : MongoDB
tags : [MongoDB]
---
{% include JB/setup %}

`MongoDB Optimizer`  

## base
MongoDB索引基础，以前调研时整理的，引用如下：

	db.post.ensureIndex({title:1})  
	create the index if it does not exist, 1 asc, -1 desc
	Compound index ，如同mysql
	So if you have an index on
	a,b,c
	you can use it query on
	a
	a,b
	a,b,c
	db.post.reIndex() 创建所有索引
	db.post.dropIndex({title:1})
	
如同mysql一样，在mysql里需要什么索引在mongodb也就需要什么索引

## issue
粘代码
	
	PRIMARY> db.system.indexes.find({ns:"mba.user.base"})
	{ "v" : 1, "key" : { "_id" : 1 }, "ns" : "mba.user.base", "name" : "_id_" }
	{ "v" : 1, "key" : { "sid" : 1, "stt" : 1, "edt" : 1 }, "ns" : "mba.user.base", "name" : "sid_1_stt_1_edt_1" }
	{ "v" : 1, "key" : { "date" : 1 }, "ns" : "mba.user.base", "name" : "date_1" }

user.base集合上建了三个索引，{ "_id" : 1 }是默认的，{ "sid" : 1, "stt" : 1, "edt" : 1 }是个联合索引，{ "date" : 1 }是后来加的。查询语句explain如下

	PRIMARY> db.user.base.find({sid:"m-20212-1",pd:1},{date:1}).sort({date:-1}).explain()
	{
	        "cursor" : "BtreeCursor date_1 reverse",
	        "nscanned" : 5370418,
	        "nscannedObjects" : 5370418,
	        "n" : 894083,
	        "millis" : 14139,
	        "nYields" : 5,
	        "nChunkSkips" : 0,
	        "isMultiKey" : false,
	        "indexOnly" : false,
	        "indexBounds" : {
	                "date" : [
	                        [
	                                {
	                                        "$maxElement" : 1
	                                },
	                                {
	                                        "$minElement" : 1
	                                }
	                        ]
	                ]
	        }
	}

安装最初的想法，这个查询应该先用{ "sid" : 1, "stt" : 1, "edt" : 1 }这个联合索引进行过滤，再用{ "date" : 1 }索引进行排序才对，但是 explain的结果说明实际上只用到了date索引，why？？？

## explain
解释  
*MongoDB chooses an index to use for a query by trying all possible indexes, and using whichever one finishes first.*  
MongoDB会根据索引来构建QueryPlan，每个可用index对应一个QueryPlan，然后并行地执行这些plan，哪个快就用哪个，并且这个结果会被缓存起来，直到执行1000次这样的查询，或者对该集合做了特定的操作，如新建一个索引。  
使用explain(true)即可发现背后的一切

	PRIMARY> db.user.base.find({sid:"m-259-0"},{date:1}).sort({date:-1}).explain(true)
	{
	        "cursor" : "BtreeCursor date_1 reverse",
	        "nscanned" : 5370418,
	        "nscannedObjects" : 5370418,
	        "n" : 4676,
	        "millis" : 8152,
	        "nYields" : 2,
	        "nChunkSkips" : 0,
	        "isMultiKey" : false,
	        "indexOnly" : false,
	        "indexBounds" : {
	                "date" : [
	                        [
	                                {
	                                        "$maxElement" : 1
	                                },
	                                {
	                                        "$minElement" : 1
	                                }
	                        ]
	                ]
	        },
	        "allPlans" : [
	                {
	                        "cursor" : "BtreeCursor sid_1_stt_1_edt_1",
	                        "indexBounds" : {
	                                "sid" : [
	                                        [
	                                                "m-259-0",
	                                                "m-259-0"
	                                        ]
	                                ],
	                                "stt" : [
	                                        [
	                                                {
	                                                        "$minElement" : 1
	                                                },
	                                                {
	                                                        "$maxElement" : 1
	                                                }
	                                        ]
	                                ],
	                                "edt" : [
	                                        [
	                                                {
	                                                        "$minElement" : 1
	                                                },
	                                                {
	                                                        "$maxElement" : 1
	                                                }
	                                        ]
	                                ]
	                        }
	                },
	                {
	                        "cursor" : "BtreeCursor date_1 reverse",
	                        "indexBounds" : {
	                                "date" : [
	                                        [
	                                                {
	                                                        "$maxElement" : 1
	                                                },
	                                                {
	                                                        "$minElement" : 1
	                                                }
	                                        ]
	                                ]
	                        }
	                },
	                {
	                        "cursor" : "BasicCursor",
	                        "indexBounds" : {
	
	                        }
	                }
	        ],
	        "oldPlan" : {
	                "cursor" : "BtreeCursor date_1 reverse",
	                "indexBounds" : {
	                        "date" : [
	                                [
	                                        {
	                                                "$maxElement" : 1
	                                        },
	                                        {
	                                                "$minElement" : 1
	                                        }
	                                ]
	                        ]
	                }
	        }
	}

oldPlan即是cache住的Optimizer优选结果。

## game over
查询还是慢啊，问题并未最终fix。  
MongoDB索引就是一颗B-Tree，例如{ "sid" : 1, "stt" : 1, "edt" : 1 }和{ "date" : 1 }是不同字段对应不同的docs，自然无法交叉使用。哪么再建一个{sid:1,date:-1}索引能否让查询更快些？试试  

	// 新建索引，在mongos上建
	mongos> db.user.base.ensureIndex({sid:1,date:-1})	
	// 验证下，getIndexes
	PRIMARY> db.user.base.getIndexes()
	[
	        {
	                "v" : 1,
	                "key" : {
	                        "_id" : 1
	                },
	                "ns" : "mba.user.base",
	                "name" : "_id_"
	        },
	        {
	                "v" : 1,
	                "key" : {
	                        "sid" : 1,
	                        "stt" : 1,
	                        "edt" : 1
	                },
	                "ns" : "mba.user.base",
	                "name" : "sid_1_stt_1_edt_1"
	        },
	        {
	                "v" : 1,
	                "key" : {
	                        "date" : 1
	                },
	                "ns" : "mba.user.base",
	                "name" : "date_1"
	        },
	        {
	                "v" : 1,
	                "key" : {
	                        "sid" : 1,
	                        "date" : -1
	                },
	                "ns" : "mba.user.base",
	                "name" : "sid_1_date_-1"
	        }
	]
	PRIMARY>
	
	// explain 
	PRIMARY> db.user.base.find({sid:"m-259-0"},{date:1}).sort({date:-1}).explain(true)
	{
	        "cursor" : "BtreeCursor sid_1_date_-1",
	        "nscanned" : 4676,
	        "nscannedObjects" : 4676,
	        "n" : 4676,
	        "millis" : 13,
	        "nYields" : 0,
	        "nChunkSkips" : 0,
	        "isMultiKey" : false,
	        "indexOnly" : false,
	        "indexBounds" : {
	                "sid" : [
	                        [
	                                "m-259-0",
	                                "m-259-0"
	                        ]
	                ],
	                "date" : [
	                        [
	                                {
	                                        "$maxElement" : 1
	                                },
	                                {
	                                        "$minElement" : 1
	                                }
	                        ]
	                ]
	        },
	        "allPlans" : [
	                {
	                        "cursor" : "BtreeCursor sid_1_date_-1",
	                        "indexBounds" : {
	                                "sid" : [
	                                        [
	                                                "m-259-0",
	                                                "m-259-0"
	                                        ]
	                                ],
	                                "date" : [
	                                        [
	                                                {
	                                                        "$maxElement" : 1
	                                                },
	                                                {
	                                                        "$minElement" : 1
	                                                }
	                                        ]
	                                ]
	                        }
	                }
	        ]
	}

对比两次explain结果，之前millis是8152，建了新索引后是13，快了不止百倍!!!  
最后{ "date" : 1 }索引不用了，可以删掉了，以免占用内存。

	mongos> db.user.base.dropIndex({date:1})
	{
	        "raw" : {
	                "shard01/mg1sh.prod.mediav.com:27018,mg2sh.prod.mediav.com:27018,mg3sh.prod.mediav.com:27018" : {
	                        "nIndexesWas" : 4,
	                        "ok" : 1
	                },
	                "shard02/mg1sh.prod.mediav.com:27028,mg2sh.prod.mediav.com:27028" : {
	                        "nIndexesWas" : 4,
	                        "ok" : 1
	                },
	                "shard03/mg1sh.prod.mediav.com:27038,mg3sh.prod.mediav.com:27038" : {
	                        "nIndexesWas" : 4,
	                        "ok" : 1
	                }
	        },
	        "ok" : 1
	}

## references
+ [Query Optimizer](http://www.mongodb.org/display/DOCS/Query+Optimizer)
+ [mongodb not using indexes when sorting?](http://stackoverflow.com/questions/8440694/mongodb-not-using-indexes-when-sorting)
+ [Explain](http://www.mongodb.org/display/DOCS/Explain#Explain-WithSharding)

