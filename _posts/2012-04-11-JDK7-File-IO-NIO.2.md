---
layout: post
title: JDK7的新文件系统(NIO.2)
category : jdk
tags : [jdk7]
---
{% include JB/setup %}

`JDK7 File IO/Featuring NIO.2`
又一片碎文

## 入口
JDK7的新文件系统主要在`java.nio.file`报下，主要入口类有

	java.nio.file.Path：文件系统中的路径，可以表示目录或文件
	java.nio.file.Paths：工具类，提供getPath工厂方法
	java.nio.file.Files：工具类，提供各种目录/文件常用操作接口

## 获取文件基本属性
代码示例如下

	URL url = NIO2.class.getClassLoader().getResource("./com/github/gengmzh/jdk7/NIO2.class");
	Path path = Paths.get(url.toURI());
	
	path.getFileName();			//获取文件名，即NIO2.class
	Files.exists(path);			//判断文件是否存在
	Files.isRegularFile(path);	//是否常规文件
	Files.isReadable(path);		//是否可读
	Files.isWritable(path);		//是否可写
	Files.isExecutable(path);	//是否可执行
	
	Map<String, Object> attr = Files.readAttributes(path, "*");	//读取文件所有属性

## 文件读写
代码示例如下

	//新建文件
	URL url = NIO2.class.getClassLoader().getResource("./com/github/gengmzh/jdk7/");
	Path path = Paths.get(url.toURI()).resolve("NIO2.class.txt");
	if (Files.notExists(path)) {
		Files.createFile(path);
	}
	
	//写文件
	String text = "File IO/NIO.2";
	Files.write(path, text.getBytes(), StandardOpenOption.WRITE);	//WRITE为重写
	Files.write(path, text.getBytes(), StandardOpenOption.WRITE);	
	
	//读文件
	text = new String(Files.readAllBytes(path));
	
	//删除文件
	Files.delete(path);

当然也可以通过Files构建InputStream/OutputStream、BufferedReader/BufferedWriter来执行读写操作。

## 文件复制和移动
代码示例如下

	URL url = NIO2.class.getClassLoader().getResource("./com/github/gengmzh/jdk7/");
	Path path = Paths.get(url.toURI()).resolve("NIO2.class.txt");
	if (Files.notExists(path)) {
		Files.createFile(path);
	}
	
	// copy
	Path target = path.resolve("../../NIO2.class.txt").normalize();
	Files.copy(path, target, StandardCopyOption.REPLACE_EXISTING);
	
	// move
	target = path.resolve("../../NIO2.class.txt").normalize();
	Files.move(path, target, StandardCopyOption.REPLACE_EXISTING);

## 目录遍历
NIO.2支持目录的先序便利，代码示例如下

	URL url = NIO2.class.getClassLoader().getResource("./com/github/gengmzh/jdk7/");
	Path path = Paths.get(url.toURI()).resolve("../").normalize();

	Files.walkFileTree(path, new FileVisitor<Path>() {

		@Override
		public FileVisitResult preVisitDirectory(Path dir, BasicFileAttributes attrs) throws IOException {
			System.out.println("dir: " + dir);
			return FileVisitResult.CONTINUE;
		}

		@Override
		public FileVisitResult visitFile(Path file, BasicFileAttributes attrs) throws IOException {
			System.out.println("file: " + file);
			return FileVisitResult.CONTINUE;
		}

		@Override
		public FileVisitResult visitFileFailed(Path file, IOException exc) throws IOException {
			exc.printStackTrace();
			return FileVisitResult.CONTINUE;
		}

		@Override
		public FileVisitResult postVisitDirectory(Path dir, IOException exc) throws IOException {
			return FileVisitResult.CONTINUE;
		}
	});


## 文件系统监控
NIO.2支持简单的文件系统监控，代码示例如下
	
	URL url = NIO2.class.getClassLoader().getResource("./com/github/gengmzh/jdk7/");
	final Path path = Paths.get(url.toURI());
	
	//监控文件/目录的创建、删除操作
	WatchService watcher = FileSystems.getDefault().newWatchService();
	WatchKey key = path.register(watcher, StandardWatchEventKinds.ENTRY_CREATE, StandardWatchEventKinds.ENTRY_DELETE);

	Thread th = new Thread() {
		public void run() {
			try {
				Path p = path.resolve("watch");
				if (Files.notExists(p)) {
					Files.createDirectory(p);
				}
				//Thread.sleep(1000);
				Files.deleteIfExists(p);
			} catch (Exception e) {
				e.printStackTrace();
			}
		};
	};
	th.start();
	th.join();

	List<WatchEvent<?>> events = key.pollEvents();
	for (WatchEvent<?> ev : events) {
		WatchEvent.Kind<?> k = ev.kind();
		Path p = (Path) ev.context();
		System.out.println(k + ": " + p);
	}


### references
+ [Java7 NIO.2](http://docs.oracle.com/javase/tutorial/essential/io/fileio.html)
+ [NIO.2完整测试代码](https://github.com/gengmzh/alg/blob/master/src/test/java/com/github/gengmzh/jdk7/NIO2.java)
