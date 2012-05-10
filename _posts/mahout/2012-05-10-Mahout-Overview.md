---
layout: post
title: Mahout Overview
category : Mahout
tags : [mahout]
---
{% include JB/setup %}
 
译自[Overview of Mahout](https://cwiki.apache.org/confluence/display/MAHOUT/Overview)

## Overview
Mahout的目标是建立一个可扩展的机器学习包，此可扩展性意在：  

+ 对较大数据集能够扩展。我们的核心算法，聚类、分类以及基于批处理的（batch-based？）协同过滤，都是在hadoop的MapReduce基础上实现的。
但是我们并不限制对Mahout的贡献都是基于hadoop的，基于单节点的或非hadoop集群的同样受欢迎。Mahout核心包都是高度优化了的，具有较好的性能，同样针对非分布式算法也进行了优化。
+ 对用户业务场景能够支持和扩展。Mahout的发布是遵守Apache软件许可证的，商业有好（commercially friendly？）。
+ 可扩展的社区。Mahout意在创建一个充满朝气的、反应灵敏的、多种多样的社区，不仅促进针对项目本身的讨论，而且包括针对潜在使用场景的。订阅我们的邮件列表吧，你会了解的更多。

<br>
当前Mahout主要支持四个使用场景：推荐，分析用户行为数据从中挖掘出用户可能感兴趣的事物；聚类，例如将文本文档分成局部相关的组；分类，从已有的已分类文档中学习属于某个具体分类的文档长什么样（look like），
据此将新的文档分到准确的类别中。常见的事物集（itemset？）挖掘会用一堆事物组（例如一次查询会话中的term、购物车中的东西），从中识别出哪些事物经常出现在一起。


## issues
简短的几句话原文读起来那么流畅，翻译起来却煞费力气，可见英文功底和专业能力都不够，得补课啦。


## references
+ [Overview of Mahout](https://cwiki.apache.org/confluence/display/MAHOUT/Overview)
