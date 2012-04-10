---
layout: post
title: hudson slave 配置
category : engineering
tags : [hudson]
---
{% include JB/setup %}

`Hudson Slave Config`

Published by gmz on 七月 4th, 2011 - in JAVA  
hudson 1.384  

## 1 slave节点配置
入口：hudson -> 系统管理 -> 管理节点，点击新建节点  
以下是需要填写的项目：  
name，slave名称  
of executors，就是并发构建的数目，建议填写服务器CPU个数  
Remote FS root，slave机器上用于hudson构建的目录，实际构建后该目录实际结构如下  

	[gmz@server01 hudson]$ ls
	jdk jdk.sh maven2.1-interceptor.jar maven-agent.jar maven-interceptor.jar slave.jar workspace

Labels，slave标记，在后面创建project会用到  
用法，尽可能的使用这个阶段/只允许运行绑定到这台机器的Job，视情况选择，这里选第一项  
Launch method，选择Launch slave agents on Unix merchines via SSH，然后点击右边的Advanced按钮，各子项如下  
host，填写slave机器ip  
username，登陆名  
password，登陆密码  
其他项默认即可  
Availability，默认keep this slave on-line as much as possible  

Node Properties中Environment variables可用于填写系统环境变量，如PATH、CLASSPATH  
名CLASSPATH，值

	.:/home/gmz/local/jdk1.6.0_25/lib

名PATH，值

	/home/gmz/local/jdk1.6.0_25/bin:/home/gmz/local/jdk1.6.0_25/jre/bin:/home/gmz/local/cmake-2.8.4/bin:/home/gmz/local/mysql-5.5.13/bin:/home/gmz/local/ant-1.8.2/bin:/usr/kerberos/bin:/usr/local/bin:/bin:/usr/bin:/sbin:/home/gmz/bin


至此，slave配置完成，点击Save按钮  
保存后等待hudson激活该slave  

## 2 在slave上创建构建任务
入口：hudson -> 新建人物 -> 填写任务名称，选择Build a free-style software project，点击OK  
Restrict where this project can be run，填写刚刚创建的slave名称  
其他各项同本地任务  

*注：Restrict where this project can be run也可填写创建slave是填写的label，此时hudson将从相同label的slave中选择一个进行构建*  

## 3 Ant的使用
先为master添加ant，入口：hudson -> 系统管理 -> 系统设置，点击新增ant，name填ant1.8.2，ant_home填写master上ant的安装目录。  
然后在slave的设置页，选中tool locations，点击add，别名选择(Ant)ant1.8.2，目录填写slave上ant的安装目录，如/home/gmz/local/ant-1.8.2。  
最后在构建任务的设置页，invoke ant下ant version选择ant1.8.2即可。  


## references
+ [Hudson CI](http://hudson-ci.org/)
