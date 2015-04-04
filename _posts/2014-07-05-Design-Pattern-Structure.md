---
layout: post
title: Design Pattern Online, structural patterns
category : Design Pattern
tags : [Design Pattern]
---
{% include JB/setup %}


`生产环境中的设计模式之结构型模式`  

结构型模式：处理类或者对象的组合，以获得更大的更加灵活的结构，并实现新的功能。主要有Adapter、Bridge、Composite、Decorator、Facade、Flyweight、Proxy等模式。  


### 1. Adapter: 适配器模式
*Convert the interface of a class into another interface clients expect. An adapter lets classes work together that could not otherwise because of incompatible interfaces.*  

**结构**  

适配器的结构图如下：  
![Adapter](/assets/images/design_pattern/adapter-class.png)  
基于继承的结构。  

![Adapter](/assets/images/design_pattern/adapter-instance.png)  
基于组合的结构。  


**解析**  

+ 适配器将不兼容的接口转换为相互的接口，即意为着对原有接口的改变。
+ 基于继承的实现更容易重载既有行为，而基于组合的实现更容易对多个adaptee进行统一适配。
+ 通过增加适配器，可以不用对不兼容的双方进行修改，也可以在适配器中适当的重定义行为，这一点和装饰模式类似。


**实例**  

实际项目中的一个适配器实现如下：  

![Adapter](/assets/images/design_pattern/adapter.x.jpg)  

+ 为了适配Worker和Processor增加了Executor类，主要是对输入、输出参数进行了转换。
+ 这里省略了适配器接口，因为可以预见无需再有其他的适配器实现，也就不用再抽象一个接口出来了。


### 2. Bridge: 桥接模式
*Decouple an abstraction from its implementation allowing the two to vary independently.*  

**结构**  

桥接模式的结构图如下：  
![Bridge](/assets/images/design_pattern/bridge.jpg)  

**解析**  


**实例**  


### 3. Composite: 组合模式
*Compose objects into tree structures to represent part-whole hierarchies. Composite lets clients treat individual objects and compositions of objects uniformly.*  

**结构**  

组合模式的结构图如下：  
![Composite](/assets/images/design_pattern/composite.jpg)  

**解析**  

+ 组合对象需继承相同的接口，并聚集其他实现类作为“整体”的“部分”，往往通过“装饰”这些“部分”的功能来实现统一接口。
+ 通过增加组合类，在不改变原有代码的情况下就实现了新的功能。

**实例**  

一个简单组合模式应用如下：  
![Composite](/assets/images/design_pattern/composite.x.jpg)  

+ 为了实现同时处理song和artist数据的功能，增加AllinfoDataParser，内部对data进行分拆，调用song和artist的parser分别处理，再将处理结果统一返回。


### 4. Decorator: 装饰模式
*Attach additional responsibilities to an object dynamically keeping the same interface. Decorators provide a flexible alternative to subclassing for extending functionality.*  

**结构**  

装饰模式的结构图如下：  
![Decorator](/assets/images/design_pattern/decorator.jpg)  

**解析**  


**实例**  


### 5. Facade: 外观模式
*Provide a unified interface to a set of interfaces in a subsystem. Facade defines a higher-level interface that makes the subsystem easier to use.*  

**结构**  

外观模式的结构图如下：  
![Facade](/assets/images/design_pattern/facade.png)  

**解析**  


**实例**  


### 6. Flyweight: 享元模式
*Use sharing to support large numbers of similar objects efficiently.*  

**结构**  

享元模式的结构图如下：  
![Flyweight](/assets/images/design_pattern/flyweight.jpg)  

**解析**  


**实例**  


### 7. Proxy: 代理模式
*Provide a surrogate or placeholder for another object to control access to it.*  

**结构**  

代理模式的结构图如下：  
![Proxy](/assets/images/design_pattern/proxy.jpg)  

**解析**  

+ 代理模式意在对客户端的访问进行控制，提供与实体对象相同的接口。而适配器意在对原有接口进行适配，往往改变了接口形态。

**实例**  

从项目中抽象出来的一个代理模式示例如下：  
![Proxy](/assets/images/design_pattern/proxy.x.jpg)  

+ 为了使得BCS操作更加安全，提供一个代理类，在get、put、delete等操作时增加重试机制。
+ 由于没有抽象接口，BCSObjectProxy直接继承了BCSObject类，并且集成了具体对象。


<br />

### references

+ [Software design pattern](http://en.wikipedia.org/wiki/Software_design_pattern)
+ [设计模式](http://baike.baidu.com/view/66964.htm)

