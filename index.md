---
layout: page
title: Programming Intelligence
tagline: stackoverflow, outofmemory, internal, unknown
---
{% include JB/setup %}

你移，或者不移  
指针就在那里，不来不去  
你放，或者不放  
内存就在那里，不舍不弃  
你爱恋，或者不爱恋  
算法就在那里，不悲，也不喜

## Blog Posts

<ul class="posts">
  {% for post in site.posts %}
    <li><span>{{ post.date | date_to_string }}</span> &raquo; <a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a></li>
  {% endfor %}
</ul>

## follows
<ul class="posts">
	<li><span>4/01 2012</span> &raquo; <a href="http://projecteuler.net/problems" target="_blank">Project Euler</a></li>
	
	<li><span>3/31 2012</span> &raquo; <a href="http://docs.oracle.com/javase/7/docs/" target="_blank">JDK7 Docs</a></li>
	<li><span>3/31 2012</span> &raquo; <a href="http://docs.oracle.com/javase/tutorial/essential/io/fileio.html" target="_blank">NIO2 File System</a></li>
	
	<li><span>3/31 2012</span> &raquo; <a href="http://mahout.apache.org/" target="_blank">Apache Mahout</a></li>
</ul>



## Sample Posts

This blog contains sample posts which help stage pages and blog data.
When you don't need the samples anymore just delete the `_posts/core-samples` folder.

    $ rm -rf _posts/core-samples

## To-Do

This theme is still unfinished. If you'd like to be added as a contributor, [please fork](http://github.com/plusjade/jekyll-bootstrap)!
We need to clean up the themes, make theme usage guides with theme-specific markup examples.
