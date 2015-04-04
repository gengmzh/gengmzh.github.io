---
layout: post
title: Design Pattern Online, behavioral patterns
category : Design Pattern
tags : [Design Pattern]
---
{% include JB/setup %}


`生产环境中的设计模式之行为型模式`  

结构型模式：对类或对象间的交互和职责分配进行描述，抽象了具体行为以实现可扩展性。
主要有Chain of responsibility、Command、Interpreter、Iterator、Mediator、Memento、Observer、State、Strategy、Template method、Visitor等模式。  


### 1. Chain of responsibility: 责任链模式
*Avoid coupling the sender of a request to its receiver by giving more than one object a chance to handle the request. Chain the receiving objects and pass the request along the chain until an object handles it.*  

**结构**  

责任链的结构图如下：  
![Responsibility](/assets/images/design_pattern/chain_of_responsibility.png)  


**解析**  


**实例**  
  

### 2. Command: 命令模式
*Encapsulate a request as an object, thereby letting you parameterize clients with different requests, queue or log requests, and support undoable operations.*  

**结构**  

命令模式的结构图如下：  
![Command](/assets/images/design_pattern/command.png)  

**解析**  


**实例**  


### 3. Interpreter: 解释器模式
*Given a language, define a representation for its grammar along with an interpreter that uses the representation to interpret sentences in the language.*  

**结构**  

解释器模式的结构图如下：  
![Interpreter](/assets/images/design_pattern/interpreter.png)  

**解析**  


**实例**  


### 4. Iterator: 迭代器模式
*Provide a way to access the elements of an aggregate object sequentially without exposing its underlying representation.*  

**结构**  

迭代器模式的结构图如下：  
![Iterator](/assets/images/design_pattern/iterator.png)  

**解析**  


**实例**  


### 5. Mediator: 媒介模式
*Define an object that encapsulates how a set of objects interact. Mediator promotes loose coupling by keeping objects from referring to each other explicitly, and it lets you vary their interaction independently.*  

**结构**  

媒介模式的结构图如下：  
![Mediator](/assets/images/design_pattern/mediator.png)  

**解析**  


**实例**  


### 6. Memento: 备忘录模式
*Without violating encapsulation, capture and externalize an object's internal state allowing the object to be restored to this state later.*  

**结构**  

备忘录模式的结构图如下：  
![Memento](/assets/images/design_pattern/memento.png)  

**解析**  


**实例**  


### 7. Observer: 观察者模式
*Define a one-to-many dependency between objects where a state change in one object results in all its dependents being notified and updated automatically.*  

**结构**  

观察者模式的结构图如下：  
![Observer](/assets/images/design_pattern/observer.png)  

**解析**  


**实例**  


### 8. State: 状态模式
*Allow an object to alter its behavior when its internal state changes. The object will appear to change its class.*  

**结构**  

状态模式的结构图如下：  
![State](/assets/images/design_pattern/state.png)  

**解析**  


**实例**  



### 9. Strategy: 策略模式
*Define a family of algorithms, encapsulate each one, and make them interchangeable. Strategy lets the algorithm vary independently from clients that use it.*  

**结构**  

策略模式的结构图如下：  
![Strategy](/assets/images/design_pattern/strategy.png)  

**解析**  


**实例**  



### 10. Template method: 模版方法模式
*Define the skeleton of an algorithm in an operation, deferring some steps to subclasses. Template method lets subclasses redefine certain steps of an algorithm without changing the algorithm's structure.*  

**结构**  

模版方法模式的结构图如下：  
![Template method](/assets/images/design_pattern/template.png)  

**解析**  


**实例**  



### 11. Visitor: 观察者模式
*Represent an operation to be performed on the elements of an object structure. Visitor lets you define a new operation without changing the classes of the elements on which it operates.*  

**结构**  

观察者模式的结构图如下：  
![Visitor](/assets/images/design_pattern/visitor.png)  

**解析**  


**实例**  




### references

+ [Software design pattern](http://en.wikipedia.org/wiki/Software_design_pattern)
+ [设计模式](http://baike.baidu.com/view/66964.htm)

