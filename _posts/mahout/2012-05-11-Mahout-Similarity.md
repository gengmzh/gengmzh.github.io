---
layout: post
title: Mahout协同过滤中相似性算法
category : Mahout
tags : [mahout, collaborative filtering, similarity]
---
{% include JB/setup %}
 
`the similarity algorithms of collaborative filtering in Mahout`

## Overview
协同过滤中的相似性算法基本有欧几里得距离、皮尔森系数、夹角余弦等，Mahout中收集了更多，如下图所示：  
ItemSimilarity  
![ItemSimilarity](https://github.com/gengmzh/gengmzh.github.com/raw/master/_includes/mahout/ItemSimilarity.jpg)  
UserSimilarity  
![UserSimilarity](https://github.com/gengmzh/gengmzh.github.com/raw/master/_includes/mahout/UserSimilarity.jpg)  

可见Mahout为User、Item分别提供了相似性借口，基本的算法都是一样的。其中example包下的是mahout-examples样例中引入的，此文不做讨论。  

两个抽象类  

+ AbstractItemSimilarity：简单实现了ItemSimilarity中的allSimilarItemIDs方法，没有其他逻辑
+ AbstractSimilarity：是similarity包中的核心类，对基本相似性算法进行了抽象实现，例如userSimilarity方法所做的工作如下图所示

![AbstractSimilarity.userSimilarity](https://github.com/gengmzh/gengmzh.github.com/raw/master/_includes/mahout/AbstractSimilarity.png)  
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

## CityBlockSimilarity
城市区快距离，即曼哈顿距离，原意为在欧几里得空间中两点在每个坐标系上的投影距离之和。在Mahout中用其衡量两个user/item间的相似性时，以计算user相似性为例，两者都评分的item距离视为0剔除不计，
只有一个user评分的item距离记为1，也就是在以只有一个user评分的item组成的n维空间中计算Manhattan距离，公式如下

	int prefs1Size = u1评分的item数;
    int prefs2Size = u2评分的item数;
    int intersectionSize = u1、u2都评分的item数;
    int distance = pref1 + pref2 - 2 * intersection;
    return 1.0 / (1.0 + distance);

*此算法没有考虑两者的评分差异，将问题进行了二元简化，直觉认为考虑评分差异并补充数据后效果会更好*

## LogLikelihoodSimilarity
对数似然性，不可言谈只能参考附件中大师佳作了。基本公式如下

	long prefs1Size = ……
    long prefs2Size = ……
    long intersectionSize =……
    long numItems = dataModel.getNumItems();
    double logLikelihood =
        LogLikelihood.logLikelihoodRatio(intersectionSize,
                                         prefs2Size - intersectionSize,
                                         prefs1Size - intersectionSize,
                                         numItems - prefs1Size - prefs2Size + intersectionSize);
    return 1.0 - 1.0 / (1.0 + logLikelihood);

## SpearmanCorrelationSimilarity
Spearman系数，类似于皮尔森系数，不同在于用评分rank的顺序进行计算，而不评分，公式不再罗列，目前也只有UserSimilarity接口下有实现。


##　others
其他的如CachingItemSimilarity、CachingUserSimilarity、GenericItemSimilarity、GenericUserSimilarity等都是对基本算法上的装饰实现，待后续用到时在仔细研究。

## Similarity in action
从给定评分数据集总计算用户1和2之间的相似性，实验代码如下

	DataModel data = new FileDataModel(new File("C:\\Users\\gmz\\AppData\\Local\\Temp\\ratings.txt"));

	UserSimilarity similarity = new PearsonCorrelationSimilarity(data);
	double value = similarity.userSimilarity(1L, 2L);
	System.out.println(value);

	similarity = new UncenteredCosineSimilarity(data);
	value = similarity.userSimilarity(1L, 2L);
	System.out.println(value);

	similarity = new EuclideanDistanceSimilarity(data);
	value = similarity.userSimilarity(1L, 2L);
	System.out.println(value);

完成代码参见[Similarity](https://github.com/gengmzh/alg/blob/master/src/main/java/com/github/gengmzh/mahout/Similarity.java)。  
输出结果为
	
	0.4166666666666694
	0.9848484848484849
	0.5694991259569396

测试数据集从 GroupLens的“1 million”转换而来，FileDataModel的数据格式为
	
	1,1193,5
	1,661,3

表示用户1对item1193的评分为5，对item661的评分为3.

## references
+ [Mahout taste](http://mahout.apache.org//taste.html)
+ [Accurate Methods for the Statistics of Surprise and Coincidence](http://citeseerx.ist.psu.edu/viewdoc/summary?doi=10.1.1.14.5962)
+ [SURPRISE AND COINCIDENCE - MUSINGS FROM THE LONG TAIL](http://tdunning.blogspot.com/2008/03/surprise-and-coincidence.html)
