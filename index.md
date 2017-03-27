---
layout: page
title: Programming Intelligence
tagline: stackoverflow, outofmemory, internal, unknown
---
{% include JB/setup %}

## Posts

<ul class="posts">
  {% for post in site.posts %}
  	{% if post.title != 'Jekyll Introduction' %}
    <li><span>{{ post.date | date: "%Y-%m-%d" }}</span> &raquo; <a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a></li>
    {% endif %}
  {% endfor %}
</ul>

## follows
<ul class="posts">
	<li><span>4/01 2012</span> &raquo; <a href="http://projecteuler.net/problems" target="_blank">Project Euler</a></li>
	
	<li><span>3/31 2012</span> &raquo; <a href="http://docs.oracle.com/javase/7/docs/" target="_blank">JDK7 Docs</a></li>
	
	<li><span>3/31 2012</span> &raquo; <a href="http://mahout.apache.org/" target="_blank">Apache Mahout</a></li>
</ul>


---

gengmzh(@)gmail.com  

