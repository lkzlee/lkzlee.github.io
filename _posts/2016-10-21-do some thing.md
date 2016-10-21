---
layout: post
title: 技术大杂烩-java工具类总结
date:   2016-10-21 14:38:00
categories: java技术
tags: 技术积累
---
*****
* TOC
{:toc}
*****
# 1. 序言

前两天做项目突然发现对于java开发的基础工具类总结很少，虽然公司里封装提供的，jdk8自带的，google提供的第三方的等等基础库，用起来很丰富，但是发现很零散，对于java常用的处理工具类总结不够透彻，这让人很伤心呐。万一以后重新搭建新项目的，基础架构库、包等等都需要重新梳理，此篇文章因此而诞生。

# 2. java时间处理工具类库
目前时间处理类比较引人注目的是joda-time类库，从官方引入一段话
~~~text
“Joda-Time provides a quality replacement for the Java date and time classes.

Joda-Time is the de facto standard date and time library for Java prior to Java SE 8. Users are now asked to migrate to java.time (JSR-310).
~~~
它目前包含的功能还算齐全，能够很容易构造转换，判断等方法。后续可以写一个demo供参考。

其实我们用的是公司内部封装的工具类，这种工具类还是很容易封装的，没什么技术难度，但是很实用，一个时间处理工具类基本需要包含以下几种方法：
	a. 方便获取时间日期类
	b. 日期的转换和格式化
	c. 日期相关逻辑判断
这些方法基本就足以完成常用日期类的用途开发。
# 3. java对象与Json转换工具类库

# 4. java对象与xml转换工具类库
# 5. java对象与对象（属性copy）转换类库
# 6. java集合类处理工具类库
# 7. java多线程并发常用工具类
# 8. java数据库连接池
# 9. java其他工具类库（MD5，RSA等加密安全工具类）