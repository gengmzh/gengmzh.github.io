---
layout: post
title: PHP, Windows下开发环境搭建
category : PHP
tags : [PHP, IDE]
---
{% include JB/setup %}


`PHP IDE on Windows`  
本文简要记录Windows下PHP开发环境搭建的步骤，以备后续自查。  

### 1. Apache httpd
从[Apache httpd官网](http://httpd.apache.org/)下载最新版本的httpd服务器，当前版本是[httpd-2.2.25-win32-x86-openssl-0.9.8y.msi](http://apache.fayea.com/apache-mirror//httpd/binaries/win32/httpd-2.2.25-win32-x86-openssl-0.9.8y.msi)。  
下载后根据提示逐步安装完成，在系统托盘处进行启停。默认端口为80，启动后直接访问`http://localhost`，显示

  It works!

即表示httpd服务器安装成功。  
关于安装版本：

+ no ssl: 不支持ssl安全认证
+ openssl: 支持ssl安全认证，使用https协议而非http。

**修正**  
为了兼容最新版本的PHP，这里需要安装[Apache Lounge](http://www.apachelounge.com/)发布的最新版的httpd，当前是[httpd-2.4.9-win32-VC11.zip](http://www.apachelounge.com/download/VC11/binaries/httpd-2.4.9-win32-VC11.zip)。  
参照压缩包中的`ReadMe.txt`进行安装，注意不要漏掉`Visual C++ Redistributable for Visual Studio 2012 Update 4`这一步，将压缩包解压到`C:\Apache24`目录。  


httpd相关配置参见[httpd docs](http://httpd.apache.org/docs/2.2/)。  


### 2. PHP及扩展服务
从[PHP官网](http://php.net/)下载最新版本的PHP，当前版本是[php-5.5.13-Win32-VC11-x86.zip](http://windows.php.net/downloads/releases/php-5.5.13-Win32-VC11-x86.zip)。  
关于版本选择请参考[PHP Download](http://windows.php.net/download/)左侧`Which version do I choose?`描述。  
将压缩包解压到`C:\php-5.5.13`目录，参照其中的`install.txt`文件进行安装，主要有：

+ 复制php.ini-development，命名为php.ini
+ 修改doc_root = "C:/Apache24/htdocs"
+ 修改extension_dir = "C:/php-5.5.13/ext"
+ 打开扩展库，如extension=php_mysql.dll、extension=php_curl.dll等
+ 修改时区date.timezone = Asia/Shanghai

**Installing as an Apache module**  
在`httpd.conf`中找到最后一个#LoadModule，在其后添加：

	# For PHP 5 do something like this:
	LoadModule php5_module "C:/php-5.5.13/php5apache2_4.dll"
	AddType application/x-httpd-php .php
	# configure the path to php.ini
	PHPIniDir "C:/php-5.5.13"


修改DirectoryIndex如下：

	<IfModule dir_module>
		DirectoryIndex index.php index.html
	</IfModule>


在htdocs目录下添加`index.php`文件，内容为：

	<?php
		phpinfo();
	?>


重启httpd，能正常输出php信息即表示PHP安装配置成功。  


### 3. PDT for Eclipse
参考[PDT安装指南](http://wiki.eclipse.org/PDT/Installation)。  
打开Help->Install New Software...，选中Juno Repository`Juno - http://download.eclipse.org/releases/juno`，在Programming Languanges下选中`PHP Development Tools(PDT)`，逐步安装即可。  
*这里有PHP Development Tools(PDT)与PHP Development Tools(PDT) SDK Feature两个选项，如果同时按照可能会出现版本冲突，先装第一个即可*  


**debugger**  
由于zend debugger尚不支持PHP 5.5，这里只能选用XDebug，当前最新版本为[php_xdebug-2.2.5-5.5-vc11.dll](http://xdebug.org/files/php_xdebug-2.2.5-5.5-vc11.dll)。  
不知道应该安装哪个版本可使用[custom installation instructions](http://xdebug.org/wizard.php)。  


### 4. PHP项目部署
在`httpd.conf`中添加如下配置，指定代码目录，重启httpd即可：  

	<IfModule dir_module>  
	    DirectoryIndex index.php index.html index.htm  
	    Alias /php-test "D:/workspace/proj/quku/php-test"  
	    <Directory D:/workspace/proj/quku/php-test>  
	    Options Indexes FollowSymLinks  
	    AllowOverride None
	    Require all granted
	    </Directory>  
	</IfModule>



### references
+ [Apache httpd](http://httpd.apache.org/)
+ [Apache Lounge](http://www.apachelounge.com/)
+ [PHP](http://php.net/)
+ [PHP Development Tools](http://projects.eclipse.org/projects/tools.pdt)
+ [xdebug](http://xdebug.org/index.php)
+ [zend debugger](http://www.zend.com/products/studio/downloads)
