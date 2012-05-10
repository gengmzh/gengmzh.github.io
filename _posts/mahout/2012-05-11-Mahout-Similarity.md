---
layout: post
title: Mahout协同过滤中相似性算法
category : Mahout
tags : [mahout, collaborative filtering, similarity]
---
{% include JB/setup %}
 
`the similarity algorithms of collaborative filtering in Mahout`

## Overview
协同过滤中的相似性算法基本有欧几里得距离、皮尔森系数、夹角余弦等，Mahout中收集了更多，相关类如下图所示：  
![ItemSimilarity](https://github.com/gengmzh/gengmzh.github.com/raw/master/_includes/ItemSimilarity.jpg)  
![UserSimilarity](https://github.com/gengmzh/gengmzh.github.com/raw/master/_includes/UserSimilarity.jpg)  

可见Mahout为User、Item分别提供了相似性借口，基本的算法都是一样的。  

两个抽象类  

+ AbstractItemSimilarity：简单实现了ItemSimilarity中的allSimilarItemIDs方法，没有其他逻辑
+ AbstractSimilarity：是similarity包中的核心类，对userSimilarity和itemSimilarity进行了抽象实现，所做的工作如下图所示

![AbstractSimilarity.userSimilarity](https://github.com/gengmzh/gengmzh.github.com/raw/master/_includes/AbstractSimilarity.png)
itemSimilarity所做工作与此类似。

*这种设计看起来比较混乱，封装的不够清楚，抽象的又有些过，可能是是一时的权宜之计，也有可能是另外有什么考虑*

## EuclideanDistanceSimilarity
欧几里得距离，以求两个user的相似性为例，假设两者对n个相同事物进行了评分，以事务为坐标以评分为坐标值，那么u1、u2就是这个坐标系中的两个点，用两点间的距离可以粗略地评估u1、u2的相似性，距离越长相似性越小，基本公式如下  

	1.0 / (1.0 + Math.sqrt(sumXYdiff2) / Math.sqrt(n))

其中`/ Math.sqrt(n)`在`programming collective intelligence`一书并没有提及，是Mahout收集的对几何距离算法的进一步优化，以克服维度增加带来的相似性偏差。

## UncenteredCosineSimilarity
夹角余弦，在上述的坐标系中，也可以两点间的夹角余弦来评估u1、u2的相似性，计算公司如下  

	sumXY/(Math.sqrt(sumX2) * Math.sqrt(sumY2))

## PearsonCorrelationSimilarity
皮尔森系统，仍以求两个user的相似性为例，假设两者对n个相同事物进行了评分，以u1、u2分别为x、y轴，以u1、u2对同一事物的评分为点，
在这个二维坐标系中拟合出一条直线使之距离各个评分点总体最近，就可以用拟合直线的角度来评估u1、u2的相似性，当斜角为45度时相似性最好。

	double meanX = sumX / count;
	double meanY = sumY / count;
	double centeredSumXY = sumXY - meanY * sumX;
	double centeredSumX2 = sumX2 - meanX * sumX;
	double centeredSumY2 = sumY2 - meanY * sumY;
	centeredSumXY/(Math.sqrt(centeredSumX2) * Math.sqrt(centeredSumY2))

## TanimotoCoefficientSimilarity
Tanimoto系数，只是简单的是与否，不考虑评分的大小，以两个用户关注的事物交集大小比上并集大小所得比值来表示相似性，公式如下

	int intersectionSize = ……
    int unionSize = xPrefsSize + yPrefsSize - intersectionSize;
    return (double) intersectionSize / (double) unionSize;

## 

## references
+ [Overview of Mahout](https://cwiki.apache.org/confluence/display/MAHOUT/Overview)
