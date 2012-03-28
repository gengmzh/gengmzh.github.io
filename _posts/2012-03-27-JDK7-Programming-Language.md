---
layout: post
title: JDK7的语法糖
category : jdk
tags : [jdk7, syntactic, sugar]
---
{% include JB/setup %}

`JDK7 Syntactic Sugar`

## binary literals
支持二进制数，使用0b或0B前缀表示，如0b00000010表示2

	byte bin = (byte) 0b00000010;
	Assert.assertEquals(bin, 2);


## underscore in number
在数值中可以随意插入下划线了，从而提升可读性，例如100_100仍表示10万

	int seq = 100_100_100;
	Assert.assertEquals(seq, 100100100);


## switch String
switch语句终于支持字符串了，示例如下

	String key = "tue";
	int week = 0;
	switch (key) {
	case "mon":
		week = 1;
		break;
	case "tue":
		week = 2;
		break;
	case "wed":
		week = 3;
		break;
	case "thr":
		week = 4;
		break;
	case "fri":
		week = 5;
		break;
	case "sat":
		week = 6;
		break;
	case "sun":
		week = 7;
		break;
	default:
		break;
	}
	Assert.assertEquals(week, 2);


## 泛型增强
在声明泛型实例时，可以只用钻石符号&lt;&gt;，省略具体的类型，例如

	List<String> list = new ArrayList<>();
	list.add("str001");
	list.add("str002");
	Assert.assertTrue(list.contains("str001"));

其次在原先泛型类、泛型方法的基础上，有支持了泛型构造子，例如

	static class Generic<T> {
		T title = null;
		Object value = null;

		<V> Generic(T t, V v) {
			title = t;
			value = v;
		}

		public T getTitle() {
			return title;
		}

		public Object getValue() {
			return value;
		}
	}
	
	Generic<String> gen = new Generic<>("g", 10000L);
	Assert.assertEquals("g", gen.getTitle());
	Assert.assertEquals(10000L, gen.getValue());

## compiler增强
在编译时针对泛型可能出现的heap pollution问题给出更严谨的警告和Error


## try-with-resource语句
在try-with-resource语句中声明的资源，并且这些资源继承了AutoCloseable接口，在语句执行完成时都会自动close，示例如下
	
	String file=...
	try (BufferedWriter writer = new BufferedWriter(new FileWriter(file))) {
		writer.write("line one");
		writer.newLine();
	}


## try catch语句增强
其一一个catch里可以捕获多个异常，示例如下

	int i = 10;
	try {
		if (i < 1) {
			throw new IllegalArgumentException("i<1");
		}
		if (i < 100) {
			throw new IllegalStateException("i<100");
		}
	} catch (IllegalArgumentException | IllegalStateException e) {
		e.printStackTrace();
	}
		
其二是throws时指定了IllegalArgumentException，方法体内可以直接抛出该异常，不用再显示cast，示例如下

	public void test_throws() throws IllegalStateException {
		int i = 10;
		try {
			if (i < 100) {
				throw new IllegalStateException("i<100");
			}
		} catch (Exception e) {
			e.printStackTrace();
			throw e;
		}
	}



### references
+ [Java7 Docs](http://docs.oracle.com/javase/7/docs/)
