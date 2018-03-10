---
layout: post
title: 技术大杂烩-java工具类总结
date:   2016-10-21 14:38:00
categories: java技术
tags: 技术积累
description: java基础库
---
*****
* TOC
{:toc}
*****

# 1. 序言

前两天做项目突然发现对于java开发的基础工具类总结很少，虽然公司里封装提供的，jdk8自带的，google提供的第三方的等等基础库，用起来很丰富，但是发现很零散，对于java常用的处理工具类总结不够透彻，这让人很伤心呐。万一以后重新搭建新项目的，基础架构库、包等等都需要重新梳理，此篇文章因此而诞生。

# 2. java时间处理工具类库

java的日期处理目前大多数基本都是手动封装的，而且手动封装的很可靠也很好用，但如果想尝试其他时间处理类的库也是有的，比如joda，

其实我们用的是公司内部封装的工具类，这种工具类还是很容易封装的，没什么技术难度，但是很实用，一个时间处理工具类基本需要包含以下几种方法：

- 方便获取时间日期类
- 日期的转换和格式化
- 日期相关逻辑判断

这些方法基本就足以完成常用日期类的用途开发。如果实在懒得自己动手封装一个常用的时间工具类可以使用joda。

joda-time类库从官方引入一段话：“Joda-Time provides a quality replacement for the Java date and time classes.Joda-Time is the de facto standard date and time library for Java prior to Java SE 8. Users are now asked to migrate to java.time (JSR-310).”

它目前包含的功能还算齐全，能够很容易构造转换，判断等方法。后续可以写一个demo供参考。




# 3. java对象与Json转换工具类库

json工具类目前是比较多的，最流行的莫过于json-lib库，总的来说分为以下几种：json-lib库，jackson库，gson库（google提供的，非常好用），fastjson（阿里巴巴工程师开发的）以后可以具体研究。

如果只用到java对象转json串，json串转对象等常用功能Gson，建议使用google提供的gson，一句话：too powerful，too simple！

对于fastjson接触不多，但是号称比较好用，性能杠杠滴，江湖传言有如下特征：

- 速度最快，测试表明，fastjson具有极快的性能，超越任其他的Java Json parser。包括自称最快的JackJson，是GSON解析速度的6倍；

- 功能强大，完全支持Java Bean、集合、Map、日期、Enum，支持范型，支持自省；无依赖，能够直接运行在Java SE 5.0以上版本；支持Android；开源 (Apache 2.0)

这种工具已经很常用了，我们公司内部之前对json-lib进行二次封装使得更加便捷好用，其实基于自身业务可以对常用的工具类进行定制化开发，以更加便捷的提高开发效率。如果没有相关定制化需要，可以尝试使用gson和fastjson，都是比较主流好用json工具，json-lib相比较而言已经上了年纪了。

从此处 [Vicky's Blog](http://vickyqi.com/2015/10/19/%E5%87%A0%E7%A7%8D%E5%B8%B8%E7%94%A8JSON%E5%BA%93%E6%80%A7%E8%83%BD%E6%AF%94%E8%BE%83/ "原文地址") 又转载了一部分描述，感觉有理有据，作者很强大很到位的对各个工具进行了性能分析：

- Gson（项目地址：https://github.com/google/gson）
  Gson是目前功能最全的Json解析神器，Gson当初是为因应Google公司内部需求而由Google自行研发而来，但自从在2008年五月公开发布第一版后已被许多公司或用户应用。Gson的应用主要为toJson与fromJson两个转换函数，无依赖，不需要例外额外的jar，能够直接跑在JDK上。而在使用这种对象转换之前需先创建好对象的类型以及其成员才能成功的将JSON字符串成功转换成相对应的对象。类里面只要有get和set方法，Gson完全可以将复杂类型的json到bean或bean到json的转换，是JSON解析的神器。

- FastJson（项目地址：https://github.com/alibaba/fastjson）
  Fastjson是一个Java语言编写的高性能的JSON处理器,由阿里巴巴公司开发。无依赖，不需要例外额外的jar，能够直接跑在JDK上。FastJson在复杂类型的Bean转换Json上会出现一些问题，可能会出现引用的类型，导致Json转换出错，需要制定引用。FastJson采用独创的算法，将parse的速度提升到极致，超过所有json库。
 
- Jackson（项目地址：https://github.com/FasterXML/jackson）
  相比json-lib框架，Jackson所依赖的jar包较少，简单易用并且性能也要相对高些。而且Jackson社区相对比较活跃，更新速度也比较快。Jackson对于复杂类型的json转换bean会出现问题，一些集合Map，List的转换出现问题。Jackson对于复杂类型的bean转换Json，转换的json格式不是标准的Json格式。
 
- Json-lib（项目地址：http://json-lib.sourceforge.net/index.html）
  json-lib最开始的也是应用最广泛的json解析工具，json-lib 不好的地方确实是依赖于很多第三方包，包括commons-beanutils.jar，commons-collections-3.2.jar，commons-lang-2.6.jar，commons-logging-1.1.1.jar，ezmorph-1.0.6.jar，对于复杂类型的转换，json-lib对于json转换成bean还有缺陷，比如一个类里面会出现另一个类的list或者map集合，json-lib从json到bean的转换就会出现问题。json-lib在功能和性能上面都不能满足现在互联网化的需求。


# 4. java对象与xml转换工具类库

主流的xml解析工具有以下几种：dom、sax。衍生出来的工具库则由以下几种：

- DOM：在现在的Java JDK里都自带了，在xml-apis.jar包里

- SAX：[http://sourceforge.net/projects/sax/](http://sourceforge.net/projects/sax/)

- JDOM：[http://jdom.org/downloads/index.html](http://jdom.org/downloads/index.html)

- DOM4J：[http://sourceforge.net/projects/dom4j/](http://sourceforge.net/projects/dom4j/)

- JAXB: [https://jaxb.java.net/guide/](https://jaxb.java.net/guide/)

# 5. java对象与对象（属性copy）转换类库

自己利用common-beanutil这个jar包谢了一个简单的常用对象对象装好的类库，在公司还是挺好用的，该类支持父类属性转换，其主要目的是利用反射机制对JavaBean的属性进行处理。
BeanUtils支持的转换类型如下：

- java.lang.BigDecimal
- java.lang.BigInteger
- boolean and java.lang.Boolean
- byte and java.lang.Byte
- char and java.lang.Character
- java.lang.Class
- double and java.lang.Double
- float and java.lang.Float
- int and java.lang.Integer
- long and java.lang.Long
- short and java.lang.Short
- java.lang.String
- java.sql.Date
- java.sql.Time
- java.sql.Timestamp

这里要注意一点，java.util.Date是不被支持的，而它的子类java.sql.Date是被支持的。因此如果对象包含时间类型的属性，且希望被转换的时候，一定要使用java.sql.Date类型。否则在转换时会提示argument mistype异常。

代码如下：

~~~java

package com.netease.lottery.base.util;

import java.lang.reflect.InvocationTargetException;
import java.util.Collections;
import java.util.List;
import java.util.stream.Collectors;

import org.apache.commons.beanutils.PropertyUtils;
import org.apache.commons.collections.CollectionUtils;

import com.netease.lottery.base.common.bean.Page;

public class PropertyTransformUtil
{
	/**
	 * 支持基本类型转换
	 * @param page
	 * @return
	 */
	public static <T, A> Page<T> transform(Page<A> page, Class targetClass)
	{
		Page<T> a = new Page<T>();
		a.setPageNo(page.getPageNo());
		a.setPageSize(page.getPageSize());
		a.setTotalCount((int) page.getTotalCount());
		a.setTotalPage(page.getTotalPage());
		a.setResult(transform(page.getResult(), targetClass));
		return a;
	}

	/**
	 * 支持基本类型转换
	 * @param list
	 * @return
	 */
	public static <T, A> List<T> transform(List<A> list, Class targetClass)
	{
		if (CollectionUtils.isEmpty(list))
			return Collections.emptyList();
		List<T> taragetList = list.stream().map(t -> {
			T target = transform(t, targetClass);
			return target;
		}).collect(Collectors.toList());
		return taragetList;
	}

	/**
	 * 支付基本类型转换
	 * @param obj
	 * @return
	 */
	public static <T, A> T transform(A obj, Class targetClass)
	{
		if (obj == null)
			return null;
		T targetObj = null;
		try
		{
			targetObj = (T) targetClass.newInstance();
			PropertyUtils.copyProperties(targetObj, obj);
		}
		catch (InstantiationException | IllegalAccessException e)
		{
			e.printStackTrace();
		}
		catch (InvocationTargetException e)
		{
			e.printStackTrace();
		}
		catch (NoSuchMethodException e)
		{
			e.printStackTrace();
		}

		return targetObj;
	}
}

~~~

# 6. java集合类处理工具类库


对于集合处理目前常用的有以下几种，如：

- Guava，取各种集合的子集，list转换等等都是比较好用的
- jdk8自带的Stream


此外还有相关StringUtils CollectionUtils，ArrayUtils等等判空处理使用也是比较方便的，他们经常在基础引用的库中存在，spring、common-collection中。

# 7. java多线程并发常用工具类

java常用的并发多线程就是jdk的Concurrent包，目前java基于AQS（AbstractQueuedSynchronizer，java并发包的所有事实现基本都是围绕它和CAS操作完成的）。他们常用的用法有:

~~~java
	private static ExecutorService commonExec = new ThreadPoolExecutor(50, 50, 5 * 60, TimeUnit.SECONDS,new LinkedBlockingQueue<Runnable>());
	private static ScheduledExecutorService scheduleExec = Executors.newScheduledThreadPool(10);
	private static ScheduledExecutorService singleExec = Executors.newSingleThreadScheduledExecutor();
~~~

把这些分装成一个util工厂类，也可以将线程的数量等等搞一个配置，动态调整，这个线程池基本已经可以满足一个强大的应用了。

# 8. java数据库连接池

线程池目前主流的有以下几种（如果想深入学习，自己去google）：

- dbcp 最基本的一种连接池，小项目用它就好，用起来很舒服、simple、配置方便。
- c3p0 是另外一个开源的连接池，在业界也是比较有名的，这个连接池可以设置最大和最小连接，连接等待时间等，基本功能都有。
- proxool
- druid 阿里提供的数据连接池目前已经是性能比较好的一款连接池工具。
- 
建议大型项目用druid ，它提供了ps cache优化，sql log打印等等，很好用，原来我们公司用的proxool，发现连接经常占满，自从换了druid连接基本杠杠滴。

# 9. java其他工具类库（MD5，RSA等加密安全工具类）

我们常用的java开发过程中需要用到AES、MD5、RSA甚至有些会用到CA证书。这些我们也可以封装工具类，让我们的签名加密变得更加通用。每一个公司的发展都会拆分去一个基础的签名服务器，虽然很基础但是很有必要，拆出来的签名服务可以满足安全支付交易的所有签名验签服务。

目前jdk本身也提供了这些操作，还有apache的common-codec。官网下载地址：[http://commons.apache.org/codec/](http://commons.apache.org/codec/) Commons项目中用来处理常用的编码方法的工具类包，例如DES、SHA1、MD5、Base64, 及 hex, metaphone, soundex 等编码演算。
