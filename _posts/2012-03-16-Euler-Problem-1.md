---
layout: post
title: Euler Problem 1
category : lessons
tags : [euler]
---
{% include JB/setup %}

`Euler第一题`

## Question
If we list all the natural numbers below 10 that are multiples of 3 or 5, we get 3, 5, 6 and 9. The sum of these multiples is 23.
Find the sum of all the multiples of 3 or 5 below 1000.  
[problem 1](http://projecteuler.net/problem=1){:target="_blank"}

## Answer
其实就是得考虑下公倍数问题，一开始还遗漏了，汗。  

考虑效率的话，可以先算出3的倍数和，在算出5的倍数和，然后再减去3和5的最小公倍数15的倍数和即可，代码如下：

    public static long sumMultipleOfThreeAndFive(long n) {
    	long sum = 0;
    	long d = (n - 1) / 3;
    	sum += 3 * d * (1 + d) / 2;
    	d = (n - 1) / 5;
    	sum += 5 * d * (1 + d) / 2;
    	d = (n - 1) / 15;
    	sum -= 15 * d * (1 + d) / 2;
		return sum;
	}

考虑程序的扩展性，比如是求因子6，7，9的倍数，需要减去6和7、7和9及6和9的公倍数，最后还要加上6、7、9三者的公倍数，程序复制性会立马上升。

一般解法，从1到n过滤一遍所有数，时间复杂度也只有O(n\*m)，其中m为因子个数。  
完整代码见：[alg](https://github.com/gengmzh/alg/blob/master/src/main/java/com/github/gengmzh/euler/Problem1.java)

