---
layout: post
title: Java并发编程, 同步问题及锁
category : jdk
tags : [jdk, concurrency]
---
{% include JB/setup %}


`Java多线程中的并发问题和锁`  

通过共享数据方式实现线程间的通信固然高效，但却可能引入线程冲突（Thread Interference）、内存一致性错误（Memory Consistency Errors）两种经典错误。
要解决这两种错误就是使用同步，而同步又有可能造成线程竞争，使得系统运行变慢，甚至出现死锁（Deadlock）、饥饿（Starvation）、活锁（Livelock）等情况。


### 一. 并发同步问题  

**1. 线程冲突**  

比如简单的计数问题，伪代码如下：  

	int c=0;
	incr: c++;
	decr: c--;

变量c存储个数，incr方法加1，decr方法减1，两个方法都没有同步.
在单线程情况下，无论按什么顺序执行incr、decr都不会造成混乱，结果都是可以预期的。
但在多线程情况下，就会出现混乱，因为自增、自减操作并非原子操作，JVM会将其拆分为读取c、加1或减1、保存c三步。以A、B两个线程为例，描述一个混乱的场景：  

+ 初始：c=0，A、B线程启动；
+ A读取c=0；
+ B读取c=0；
+ A执行incr操作，结算结果=1；
+ B执行decr操作，计算结果=-1；
+ A保存结果，得c=1；
+ B保存结果，得c=-1；

可以看出结果并非正常预期的0，且只要B快点后两步变换下顺序，最终结果就为1，完全不可预测，在实际项目中这种错误将非常难于排查。  
这就是线程冲突，解决办法很简单就是同步（synchronized）或加锁（lock）。  


**2. 内存一致性错误**  

内存一致性错误，也可叫做“内存冲突”，就是脏读现象，一个线程的写操作对于另一个线程的读操作并不可见。极简示例：  

	int c=0;
	s1: c=1;
	s2: print c;

初始变量c=0，s1第一步设置c=1，s2第二部输出c。
在单线程情况下，输出结果必然是1.
但在多线程情况下，结果就未必。比如A、B两个线程，A执行s1，然后B执行s2，输出结果有可能还是0.因为JVM为了提升性能会在栈中存储c的拷贝，在B执行print的时候，A虽然执行了c=1但可能还没有回写到堆中。  
这就是内存一致性错误，解决办法有两个：  

+ volatile：说明c为volatile变量，使得线程A、B不会在栈中保存c的拷贝，读写直接操作堆，在这个极简示例中应该是可以保证A的更新对B及时空间。但如果s1稍微复杂点，比如改成c++就有可能再出现`线程冲突`问题。
+ 同步：s1、s2是同步的，那么执行s1时就会从堆中读入c，计算后回写堆，此时s2执行从堆中读入c再输出，既不会出现`内部一致性`问题，也不会出现`线程冲突`问题。  


**3. 死锁问题**  

死锁问题产生的根源是线程之间相互等待，必须需要锁住对方的资源，而对方有不释放资源。示例如下：  

	Object la, lb;
	
	t1:
	synchronized(la){
		synchronized(lb){
			// do something
		}
	}
	
	t2:
	synchronized(lb){
		synchronized(la){
			// do something
		}
	}

这里如果t1、t2两个线程按下列顺序执行，就会出现死锁。  

+ t1锁定la；
+ t2锁定lb；
+ t1请求获得lb，而lb已被t2锁定，因此t1进入等待；
+ t2请求获得la，而la已被t1锁定，因此t2金融等待；

如此两个线程“死锁”，死锁有两个解决办法，一个是让t1、t2按相同的顺序去获得锁，避免出现一个锁的环，另一个就是加大锁的粒度，减少锁的数量。  
*注：参考文档《Java 多线程面试问题汇总》中提到的使用Lock方法，实际上也是加大了锁的粒度，没有本质区别。*  


**4. 饥饿问题**  

当一个线程长时间获得不到资源，总是被其他线程抢占，那么这个线程就一直处于等待状态，这就是饥饿问题。  
要解决饥饿问题就要用到公平锁，具体参见第二部分描述。  


**5. 活锁问题**  

如果两个线程过于友好，比如线程A总是礼让线程B，让B优先获得锁，而B也一样，那么A、B就有可能不停地进行工作切换。  
这就是活锁，活锁有一定几率可以自行解开，而死锁一定是无法自解的。要避免活锁，只需要优化“礼让”的问题，比如使用先到先得的策略等。  


## 二. Java中的锁及其他

**1. 同步：synchronized**  

synchronized关键字简洁明了，可以用来声明同步方法，也可以同步代码块，两者没有本质区别，只是同步代码块时粒度可以控制得更细些，效率也就更高。  
同步方法是和当前对象实例关联的，也就是需要获取this的锁，而同步的静态方法是和当前类实例关联的，也就是需要获得class的锁。  

synchronized是在JVM层面实现的同步，Java中每个对象都有一个监视器（Monitor Lock），或叫本质锁（Intrinsic Lock）。进入synchronized方法或代码块时，需要先获得响应对象的监视器，退出时再释放监视器。  

+ synchronized同步是可重入的；
+ synchronized同步是排他的，同一时间只有一个线程可以后的监视器，即原子性；
+ 执行synchronized方法或代码块时，会先清除栈中数据从堆中重新读入，执行完后再将最新结果回写到堆中，即可见性；


**2. 锁：Lock**  

JDK1.5在java.util.concurrent.locks包下提供了另一套锁机制，Lock完全在jdk api层面实现，非常精巧，这里先简列要点.  
先看下Lock接口常用方法：  

+ void lock()：获得锁，如果锁不可用则一直等待直到获得为止；
+ boolean tryLock()：如果lock成功立即返回true，否则返回false；如果指定时间，则在指定时间获得锁则反馈true，否则返回false；
+ void lockInterruptibly()：如果lock住直接返回，否则等待直到获得锁，可以被其他线程interrupt；

Lock的唯一实现类就是ReentrantLock，具备可重入性，默认是非公平锁。公平锁、非公平锁含义如下：  

+ 公平锁：尽量按照请求锁的顺序来获得锁，当锁被释放时，等待时间最久的线程获得该锁；
+ 非公平锁：不保证按照请求锁的顺序来分配锁，所有等待线程随机抢占；

此外还有ReadWriteLock接口，用于分拆读写操作，使得多个线程可以同时读，而同一时间点只有一个线程可以写。ReentrantReadWriteLock是其唯一实现。  


**3. synchronized、Lock对比**  

两者对比异同点如下：  

+ 两者都可重入；
+ synchronized自动释放占有的锁，即便在发生异常的时候；Lock需要显示地unlock；
+ synchronized不可中断，Lock可以中断（lockInterruptibly方法），可以知道是否获得了锁（tryLock方法），因而可以做回退操作；
+ Lock可以支持读写锁，提升读操作性能；


### 
### references

+ [The Java™ Tutorials: Concurrency](http://docs.oracle.com/javase/tutorial/essential/concurrency/sync.html)
+ [Java 多线程面试问题汇总](http://www.ituring.com.cn/article/111835)
+ [深入浅出 Java Concurrency](http://blog.csdn.net/fg2006/article/details/6397900)
+ [深入JVM锁机制](http://wenku.baidu.com/view/41480552f01dc281e53af090.html)
+ [Java并发编程：Lock](http://www.cnblogs.com/dolphin0520/p/3923167.html)

