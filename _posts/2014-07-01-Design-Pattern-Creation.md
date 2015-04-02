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
![Bloom Filter](https://github.com/gengmzh/gengmzh.github.com/raw/master/_includes/design_pattern/abstract_factory.jpg)  

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
![Bloom Filter](https://github.com/gengmzh/gengmzh.github.com/raw/master/_includes/design_pattern/factory_method.png)  

**解析**  

+ 


**实例**  


### 3. Builder: 建造者模式
*Separate the construction of a complex object from its representation, allowing the same construction process to create various representations.*  

**结构**  

建造者的结构图如下：  
![Bloom Filter](https://github.com/gengmzh/gengmzh.github.com/raw/master/_includes/design_pattern/builder.png)  

**解析**  

+ 


**实例**  


### 4. Prototype: 原型模式
*Specify the kinds of objects to create using a prototypical instance, and create new objects by copying this prototype.*  

**结构**  

原型模式的结构图如下：  
![Bloom Filter](https://github.com/gengmzh/gengmzh.github.com/raw/master/_includes/design_pattern/prototype.jpg)  

**解析**  

+ 


**实例**  


### 5. Singleton: 单例模式
*Ensure a class has only one instance, and provide a global point of access to it.*  

**结构**  

工厂方法的结构图如下：  
![Bloom Filter](https://github.com/gengmzh/gengmzh.github.com/raw/master/_includes/design_pattern/singleton.png)  

**解析**  

+ 


**实例**  




### references

+ [Software design pattern](http://en.wikipedia.org/wiki/Software_design_pattern)
+ [设计模式](http://baike.baidu.com/view/66964.htm)
