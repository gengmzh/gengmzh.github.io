---
layout: post
title: Inside Hadoop, Hadoop IPC/RPC
category : java
tags : [hadoop, IPC, RPC]
---
{% include JB/setup %}


`IPC/RPC of Hadoop 1.0.3`  
IPC: InterProcess Communication，即进程间通信，是一个快速的轻量级的RPC框架，不使用Sun标准的序列化机制，而需要使用者自己序列化，在Hadoop里就是用其io框架。  

### 1. IPC应用
在Hadoop之外，在我们自己的项目里，能不能使用Hadoop IPC？当然是可以的，且非常简洁只需要三步。  
**a. 定义自己的Protocol**  
继承`VersionedProtocol`，定义具体的业务方法即可，如下了一个计数器接口，目前只支持加法计算。

	public interface CalculatorProtocol extends VersionedProtocol {

		public long add(long num1, long num2);

	}


*VersionedProtocol意在可以兼容不同版本，具体逻辑需要Server端自己实现*  
关于返回结果，只能是以下类型：

+ a primitive type, boolean, byte, char, short, int, long, float, double, or void; or 
+ a String; or
+ a Writable; or 
+ an array of the above types 


**b. 实现自己的Server**  
只要事先第一步定义的Protocol接口即可，如下

	public class CalculatorImpl implements CalculatorProtocol  { 

		@Override
		public long getProtocolVersion(String protocol, long clientVersion) throws IOException {
			return 0;
		}

		@Override
		public long add(long num1, long num2) {
			return num1 + num2;
		}

	}

*这里默认协议版本为0，没有对不同版本做处理。*  
然后使用该协议启动Server即可，如下

	Server server = RPC.getServer(new CalculatorImpl(), "localhost", 9000, 1, true, new Configuration());
	server.start();


**c. 使用Client请求Server**  
Client直接使用就行，如下

	CalculatorProtocol protocol = (CalculatorProtocol) RPC.getProxy(CalculatorProtocol.class, 0,
			new InetSocketAddress("localhost", 9000), new Configuration());
	long result = protocol.add(10, 3);
	System.out.println("10 + 3 = " + result);


运行后，server log如下：

	03/07 15:13:35 IPC Server Responder INFO  ipc.Server       :598 - IPC Server Responder: starting
	03/07 15:13:35 IPC Server handler 0 on 9000 INFO  ipc.Server       :1358 - IPC Server handler 0 on 9000: starting
	03/07 15:13:35 IPC Server listener on 9000 INFO  ipc.Server       :434 - IPC Server listener on 9000: starting
	03/07 15:13:35 IPC Server handler 0 on 9000 INFO  ipc.RPC          :601 - Call: getProtocolVersion(com.baina.beluga.support.Hadoo...
	03/07 15:13:35 IPC Server handler 0 on 9000 INFO  ipc.RPC          :601 - Return: 0
	03/07 15:13:35 IPC Server handler 0 on 9000 INFO  ipc.RPC          :601 - Call: add(10, 3)
	03/07 15:13:35 IPC Server handler 0 on 9000 INFO  ipc.RPC          :601 - Return: 13

client log如下：

	10 + 3 = 13


### 2. IPC原理
Server端主要有三个线程：

+ Listener: 负责监听客户端请求，接受到远程调用后建立连接，并封装为一个Server.Call存入callQueue。
+ Handler: 从callQueue中take到调用后，通过反射执行Protocol实例中对应业务方法，并将结果存入调用对应的responseQueue中，如果响应队列当前只有一个返回结果则同步发送结果给Client
+ Responder: 从所有连接的responseQueue中获取调用结果，异步返回给Client

代码简析如下图  
![Bloom Filter](https://github.com/gengmzh/gengmzh.github.com/raw/master/_includes/hadoop_ipc_server.jpg)  

<br>
Client主要通过动态代理构造Protocol，将调用的业务方法及参数封装为一个Client.Call发送给Server，异步接收调用结果并通过Writable实现类进行解析。  

代码简析如下图  
![Bloom Filter](https://github.com/gengmzh/gengmzh.github.com/raw/master/_includes/hadoop_ipc_client.jpg)  



### references
+ [Hadoop Wiki IPC](http://wiki.apache.org/hadoop/ipc)
+ [在分布式应用程序中使用Hadoop IPC/RPC](http://www.cnblogs.com/gpcuster/archive/2009/09/06/1561423.html)

