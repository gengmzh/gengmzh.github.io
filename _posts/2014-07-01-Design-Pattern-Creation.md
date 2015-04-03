---
layout: post
title: Design Pattern Online, creational patterns
category : Design Pattern
tags : [Design Pattern]
---
{% include JB/setup %}


`生产环境中的设计模式之创建型模式`  

创建型模式：与对象的创建有关，抽象了实例化过程，使得系统和如何创建、组合和表示对象相独立。主要有Abstract Factory、Builder、Factory Method、Prototype、Singleton等模式。  


### 1. Abstract Factory: 抽象工厂模式
*Provide an interface for creating families of related or dependent objects without specifying their concrete classes.*  

**结构**  

抽象工厂的结构图如下：  
![Bloom Filter](/assets/images/design_pattern/abstract_factory.jpg)  

**解析**  

+ 抽象了多种产品的创建机制，不同工厂实现类创建不同簇的产品；
+ 通过封装产品的创建过程，使得对象实例化和客户端、产品表示相解耦；
+ 新增一个产品簇时，只要新增一个工厂实现类（ConcreteFactory3），其他工厂不用更改，满足OCP原则；但增加新种类产品时（ProductC），工厂接口及实现类都需要修改，不满足OCP原则；
+ 项目中工厂一般通过单例形式实现；


**实例**  
  

### 2. Factory Method: 工厂方法模式
*Define an interface for creating a single object, but let subclasses decide which class to instantiate. Factory Method lets a class defer instantiation to subclasses.*  

**结构**  

工厂方法的结构图如下：  
![Bloom Filter](/assets/images/design_pattern/factory_method.png)  

**解析**  

+ 当增加新产品的时候，只需要添加ConcreteProduct2及对应的ConcreteCreator2，满足OCP原则；
+ Client端具体要用哪个产品，可以通过一个预定义的参数来确定；


**实例**  
简化版的工厂方法模式在实际项目中的应用如下，也称简单工厂模式。  
![Factory Method](/assets/images/design_pattern/factory_method.x.jpg)  

+ 这里省列了工厂接口（或抽象类），只保留工厂实现类。
+ 客户端通过type参数告知当前要获取的是哪个Handler，即产品类型的选择交由工厂控制。
+ 这种简单实现破坏了OCP原则，在新增Handler时HandlerFactory必须跟着修改。所幸在这个系统中，Handler短期内不可能增加，简单的实现反而避免了代码臃肿。


### 3. Builder: 建造者模式
*Separate the construction of a complex object from its representation, allowing the same construction process to create various representations.*  

**结构**  

建造者的结构图如下：  
![Bloom Filter](/assets/images/design_pattern/builder.png)  

**解析**  

+ 


**实例**  


### 4. Prototype: 原型模式
*Specify the kinds of objects to create using a prototypical instance, and create new objects by copying this prototype.*  

**结构**  

原型模式的结构图如下：  
![Bloom Filter](/assets/images/design_pattern/prototype.jpg)  

**解析**  

+ 


**实例**  


### 5. Singleton: 单例模式
*Ensure a class has only one instance, and provide a global point of access to it.*  

**结构**  

工厂方法的结构图如下：  
![Bloom Filter](/assets/images/design_pattern/singleton.png)  

**解析**  

+ 单例模式有多种实现，如饿汉式、饱汉式、延迟实例化等。
+ 单例通常可以保证系统中只有一个实例，但也有例外情况需要注意，如多ClassLoader、分布式环境等。
+ 单例模式的延伸是多例，即为不同的业务场景创建多个实例。


**实例**  
实际项目中的配置类常用单例形式实现，示例如下：  


	public class Config {
		
		private static Config instance;
		
		public static Config getInstance() {
			if (instance == null) {
				synchronized (Config.class) {
					if (instance == null) {
						instance = new Config();
					}
				}
			}
			return instance;
		}
		
		private Config() {
			……
		}
		……
	}


<br />

### references

+ [Software design pattern](http://en.wikipedia.org/wiki/Software_design_pattern)
+ [设计模式](http://baike.baidu.com/view/66964.htm)

