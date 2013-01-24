---
layout: post
title: Using Django
category : hadoop
tags : [python, Django]
---
{% include JB/setup %}


`python, Django, Nginx, uwsgi on Ubuntu`  

### 1. install Nginx
using apt-get to install nginx, so easy

	$sudo apt-get update
	$sudo apt-get install nginx
	$nginx -V
	nginx version: nginx/1.1.19
	......


### 2. install uwsgi

	$sudo apt-cache search uwsgi
	uwsgi - fast, self-healing application container server
	uwsgi-app-integration-plugins - plugins for integration of uWSGI and application
	uwsgi-core - fast, self-healing application container server (core)
	
	$sudo apt-get install uwsgi
	$uwsgi --version
	uWSGI 1.0.3-debian
	
	$sudo apt-get install uwsgi-plugin-python
	$sudo apt-get install libxml2-dev
	

`libxml2-dev` is required to parse the uwsgi.xml.
also remember to install `uwsgi-plugin-python`, uwsgi using it to run python.


### 3. install Django
python is installed by default on ubuntu and other linux
download the latest Django from [Django-1.4.3](https://www.djangoproject.com/download/1.4.3/tarball/)

	tar xzvf Django-1.4.3.tar.gz
	cd Django-1.4.3
	sudo python setup.py install
	python
	>>> import django
	>>> django.get_version()
	'1.4.3'

download pymongo from [pymongo-2.4.2](http://pypi.python.org/packages/source/p/pymongo/pymongo-2.4.2.tar.gz#md5=102a00761067d0c0a6b91f33840d811e)
 since the app using it.

	$tar xzvf pymongo-2.4.2.tar.gz 
	$cd pymongo-2.4.2/
	$sudo python setup.py install
	$python
	Python 2.7.3 (default, Aug  1 2012, 05:14:39) 
	[GCC 4.6.3] on linux2
	Type "help", "copyright", "credits" or "license" for more information.
	>>> import pymongo
	>>> pymongo.version
	'2.4.2'


### 4. config app & run it
the last step is to config django app and nginx

	$cat uwsgi.xml 
	<uwsgi>
	    <socket>:8087</socket>
	    <pythonpath>/var/app/href/api</pythonpath>
	    <module>wsgi</module>
	    <processes>1</processes>
	    <master>true</master>
	    <pidfile>/var/log/href/api/uwsgi.pid</pidfile>
	    <logdate>true</logdate>
	    <daemonize>/var/log/href/api/uwsgi.log</daemonize>
	    <harakiri>20</harakiri>
	    <harakiri-verbose/>
	    <memory-report/>
	    <post-buffering>8192</post-buffering>
	    <post-buffering-bufsize>65536</post-buffering-bufsize>
	</uwsgi>

pythonpath is where your wsgi.py located

	$ cat /etc/nginx/sites-enabled/href-api 
	server {
	        listen   80; ## listen for ipv4; this line is default and implied
	        #listen   [::]:80 default ipv6only=on; ## listen for ipv6
	
	        #root /usr/share/nginx/www;
	        #index index.html index.htm;
	
	        # Make site accessible from http://localhost/
	        server_name www.example.com;
	        access_log /var/log/href/api/access.log;
	        error_log /var/log/href/api/error.log;
	
	        location / {
	                root /var/app/href/api;
	                uwsgi_pass 127.0.0.1:8087;
	                include uwsgi_params;
	
	                # First attempt to serve request as file, then
	                # as directory, then fall back to index.html
	                # try_files $uri $uri/ /index.html;
	                # Uncomment to enable naxsi on this location
	                # include /etc/nginx/naxsi.rules
	        }
	}

uwsgi_pass to the port where u defined in uwsgi.xml.  
now start

	$sudo service nginx restart
	$uwsgi -x /var/app/href/api/uwsgi.xml --plugins=python
	
	$tailf /var/log/href/api/uwsgi.log 
	Thu Jan 24 18:43:29 2013 - uwsgi socket 0 bound to TCP address :8087 fd 4
	Thu Jan 24 18:43:29 2013 - Python version: 2.7.3 (default, Aug  1 2012, 05:25:23)  [GCC 4.6.3]
	Thu Jan 24 18:43:29 2013 - Python main interpreter initialized at 0x1381a80
	Thu Jan 24 18:43:29 2013 - your server socket listen backlog is limited to 100 connections
	Thu Jan 24 18:43:29 2013 - *** Operational MODE: single process ***
	Thu Jan 24 18:43:29 2013 - added /var/app/href/api/ to pythonpath.
	Thu Jan 24 18:43:29 2013 - WSGI application 0 (mountpoint='') ready on interpreter 0x1381a80 pid: 19773 (default app)
	Thu Jan 24 18:43:29 2013 - *** uWSGI is running in multiple interpreter mode ***
	Thu Jan 24 18:43:29 2013 - spawned uWSGI master process (pid: 19773)
	Thu Jan 24 18:43:29 2013 - spawned uWSGI worker 1 (pid: 19774, cores: 1)
	.....

`--plugins=python` is required, otherwise u will crash

	Thu Jan 24 18:03:02 2013 - spawned uWSGI worker 1 (pid: 17072, cores: 1)
	Thu Jan 24 18:04:33 2013 - -- unavailable modifier requested: 0 --
	Thu Jan 24 18:08:12 2013 - -- unavailable modifier requested: 0 --
	Thu Jan 24 18:08:13 2013 - -- unavailable modifier requested: 0 --
	Thu Jan 24 18:08:20 2013 - -- unavailable modifier requested: 0 --

if u have run uwsgi already, u have to kill it first

	$sudo killall -9 uwsgi
	$sudo ps aux |grep uwsgi
	# must return nothing


now bingo~


### references
+ [Nginx+uWSGI 部署 Django 应用](http://www.oschina.net/question/54100_30386)
+ [How to get Django](https://www.djangoproject.com/download/)
+ [nginx + uwsgi: unavailable modifier requested](http://stackoverflow.com/questions/10748108/nginx-uwsgi-unavailable-modifier-requested-0)

