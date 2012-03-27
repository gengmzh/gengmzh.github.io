---
layout: post
title: JDK7新增了双枢快速排序
category : jdk
tags : [jdk7, quicksort]
---
{% include JB/setup %}

`Dual Pivot Quicksort`

## classic quicksort
经典快排的思路是：  
- 选择一个中枢点P
- 将数组按中枢点分割为左右两份，左边的都小于P，右边的都大于P
- 按上述逻辑递归处理左右两个子数组

[代码示例](https://github.com/gengmzh/alg/blob/master/src/main/java/com/github/gengmzh/alg/sort/QuickSort.java)

## dual-pivot quicksort
双枢快排的思路是：  
- 选择两个中枢点P1和P2，P1<=P2
- 将数组分割为&lt;P1、&gt;P1 &amp; &lt;P2、&gt;P2三部分
- 按上述逻辑递归处理三个子数组

[代码示例](https://github.com/gengmzh/alg/blob/master/src/main/java/com/github/gengmzh/alg/sort/DualPivotQuicksort.java)

**性能比较**  
It is proved that for the Dual-Pivot Quicksort the average number of comparisons is 2*n*ln(n), the average number of swaps is 0.8*n*ln(n), 
whereas classical Quicksort algorithm has 2*n*ln(n) and 1*n*ln(n) respectively.



### references
+ [Replacement of Quicksort in java.util.Arrays with new Dual-Pivot Quicksort](http://permalink.gmane.org/gmane.comp.java.openjdk.core-libs.devel/2628)
+ [Dual Pivot Quicksort in Java](http://anshu-manymoods.blogspot.com/2009/10/dual-pivot-quicksort-in-java.html)

