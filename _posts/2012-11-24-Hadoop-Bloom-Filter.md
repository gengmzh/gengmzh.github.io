---
layout: post
title: Inside Hadoop: Bloom Filter
category : hadoop
tags : [hadoop, bloom, filter]
---
{% include JB/setup %}

`Bloom Filter`  
布隆过滤器采用hash算法将元素映射到位数租的1位或n位上，被映射到的位都置为1，没映射到的都位是0，因此判断元素是否属于集合时只需要判断其对应的位是否都为1即可，
无论是时间复杂度还是空间复杂度都是线性的，简洁而高效。当然这是有代价的：  
![Bloom Filter](https://github.com/gengmzh/gengmzh.github.com/raw/master/_includes/bloom_filter.jpg)  
集合中的元素a对应第1位、第2位，元素b对应第2位、第3位，这3位都置为1了。此时再判断元素c，由于其hash映射到第1位和第3位，就会发生碰撞，给出错误的“在集合中”的结果（false positive）。
但永远不会给出错误的“不在集合中”的判断（false negative），比如元素d映射到第3位和第5位，由于第5位为0所以肯定不在集合中。


#### 1. Filter in Hadoop
Hadoop对Bloom Filter进行了简单的抽象和封装，如下：  
![Hadoop Bloom Filter](https://github.com/gengmzh/gengmzh.github.com/raw/master/_includes/hadoop_bloom_filter.png)  

`Filter`: 抽象的过滤器父类，对基础熟悉和方法进行了抽象，属性如vectorSize、nbHash、hashType，方法如add、membershipTest、and、or、xor等。  
`BloomFilter`: 布隆过滤器的典型实现，只能插入、查找，不能删除。  
`CountingBloomFilter`: 计数过滤器将布隆过滤器中每个bit扩展为一个很小的计数器，每个counter4个bits，最大计数为15，add时一直累加直到15，delete时一直减少直到0.
4位表示一个counter并非偶然，而是数理推导的结果。  
`DynamicBloomFilter`: 从设计模式上说动态过滤器只是对布隆过滤器的简单装饰，初始时就是一个BloomFilter，当添加的元素超过设定的阀值时就再加一个BloomFilter，如此以往实现动态效果。  
`RetouchedBloomFilter`: 是对BloomFilter的扩展，通过记录每次add的元素实现有选择的清0，相比之下空间复杂度会很高，没有想到特别好的应用场景。  
  
主要属性说明：  

+ vectorSize: 位数租大小，同时也是hash算法返回的最大值。
+ nbHash: hash次数，每次求hash时返回的值个数。
+ hashType: hash函数类型，0-JenkinsHash，1-MurmurHash。


主要方法说明：

+ add: 添加元素
+ membershipTest: 判断元素是否在集合中
+ and: 两个过滤器进行与操作，即在两个集合中都出现的才记为1.
+ or: 两个过来签进行或操作，即只要在任意一个集合中出现了就记为1.
+ xor: 两个过滤器进行异或操作，即在一个集合中没出现而在另一个集合中出现的才记为1.

*CountingBloomFilter.buckets2words: 通过先减1再除以除数再加1来达到Math.ceil的效果，是个很巧妙的计算。*

#### 2. how to use?
`Bloom Filter`的应用场景很多，特别是在海量数据处理方面，比如简单的过滤、去重统计、数据抽样等。  
**独立访客统计**  

	// 统计逻辑
	userCount=0;
	BloomFilter bloomFiler=new ...
	
	public void countUser(userId)
		if(!bloomFilter.membershipTest(userId))
			userCount++;
			bloomFilter.add(userId);


**Hadoop Join**
在Hadoop分布式计算平台上，通常Join操作有两种实现方式：一是map端精确匹配，二是reduce端精确匹配。第一种方案需要把所有数据都加在到内存里，内存要求较大；第二种需要把所有数据从map传输到reducer，IO、带宽消耗较大。  
这是就可以用到Bloom Filter，方法也很简单，可以先在local环境下把所需key压缩到Bloom Filter里，然后在map阶段进行大范围的删选，可以过滤到绝大部分不需要的记录，如果是0错误率场景再在reduce阶段进行精确匹配，过滤掉那些漏网之鱼。
与前两种方式对比，节省下map阶段的内存和IO，节省了reduce阶段的IO和带宽。


#### references
+ [Bloom Filter](http://en.wikipedia.org/wiki/Bloom_filter)
+ [Counting Bloom Filter](http://wenku.baidu.com/view/f30e3945a8956bec0975e3fa.html)
+ [Theory and Network Applications of Dynamic Bloom Filters](http://www.cse.fau.edu/~jie/research/publications/Publication_files/infocom2006.pdf)
+ [Retouched Bloom Filters: Allowing Networked Applications to Trade Off Selected False Positives Against False Negatives](http://www-rp.lip6.fr/site_npa/site_rp/_publications/740-rbf_cameraready.pdf)
+ [Bloom filter inside Map Reduce](http://vanjakom.wordpress.com/2011/09/17/bloom-filter-inside-map-reduce/)

