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

	方便获取时间日期类

	日期的转换和格式化

	日期相关逻辑判断

这些方法基本就足以完成常用日期类的用途开发。


# 3. java对象与Json转换工具类库

json工具类目前是比较多的，最流行的莫过于json-lib库，总的来说分为以下几种：json-lib库，jackson库，gson库（google提供的，非常好用），fastjson（阿里巴巴工程师开发的）以后可以具体研究。

如果只用到java对象转json串，json串转对象等常用功能Gson，建议使用google提供的gson，一句话：too powerful，too simple！

对于fastjson接触不多，但是号称比较好用，性能杠杠滴，江湖传言有如下特征：

	速度最快，测试表明，fastjson具有极快的性能，超越任其他的Java Json parser。包括自称最快的JackJson，是GSON解析速度的6倍；

	功能强大，完全支持Java Bean、集合、Map、日期、Enum，支持范型，支持自省；无依赖，能够直接运行在Java SE 5.0以上版本；支持Android；开源 (Apache 2.0)

这种工具已经很常用了，我们公司内部之前对json-lib进行二次封装使得更加便捷好用，其实基于自身业务可以对常用的工具类进行定制化开发，以更加便捷的提高开发效率。如果没有相关定制化需要，可以尝试使用gson和fastjson，都是比较主流好用json工具，json-lib相比较而言已经上了年纪了。

从此处(http://vickyqi.com/2015/10/19/%E5%87%A0%E7%A7%8D%E5%B8%B8%E7%94%A8JSON%E5%BA%93%E6%80%A7%E8%83%BD%E6%AF%94%E8%BE%83/) 又转载了一部分描述，感觉有理有据，作者很强大很到位的对各个工具进行了性能分析：

	Gson（项目地址：https://github.com/google/gson）
	Gson是目前功能最全的Json解析神器，Gson当初是为因应Google公司内部需求而由Google自行研发而来，但自从在2008年五月公开发布第一版后已被许多公司或用户应用。Gson的应用主要为toJson与fromJson两个转换函数，无依赖，不需要例外额外的jar，能够直接跑在JDK上。而在使用这种对象转换之前需先创建好对象的类型以及其成员才能成功的将JSON字符串成功转换成相对应的对象。类里面只要有get和set方法，Gson完全可以将复杂类型的json到bean或bean到json的转换，是JSON解析的神器。

	FastJson（项目地址：https://github.com/alibaba/fastjson）
	Fastjson是一个Java语言编写的高性能的JSON处理器,由阿里巴巴公司开发。无依赖，不需要例外额外的jar，能够直接跑在JDK上。FastJson在复杂类型的Bean转换Json上会出现一些问题，可能会出现引用的类型，导致Json转换出错，需要制定引用。FastJson采用独创的算法，将parse的速度提升到极致，超过所有json库。

	Jackson（项目地址：https://github.com/FasterXML/jackson）
	相比json-lib框架，Jackson所依赖的jar包较少，简单易用并且性能也要相对高些。而且Jackson社区相对比较活跃，更新速度也比较快。Jackson对于复杂类型的json转换bean会出现问题，一些集合Map，List的转换出现问题。Jackson对于复杂类型的bean转换Json，转换的json格式不是标准的Json格式。

	Json-lib（项目地址：http://json-lib.sourceforge.net/index.html）
	json-lib最开始的也是应用最广泛的json解析工具，json-lib 不好的地方确实是依赖于很多第三方包，包括commons-beanutils.jar，commons-collections-3.2.jar，commons-lang-2.6.jar，commons-logging-1.1.1.jar，ezmorph-1.0.6.jar，对于复杂类型的转换，json-lib对于json转换成bean还有缺陷，比如一个类里面会出现另一个类的list或者map集合，json-lib从json到bean的转换就会出现问题。json-lib在功能和性能上面都不能满足现在互联网化的需求。



# 4. java对象与xml转换工具类库



# 5. java对象与对象（属性copy）转换类库

# 6. java集合类处理工具类库

# 7. java多线程并发常用工具类

# 8. java数据库连接池

# 9. java其他工具类库（MD5，RSA等加密安全工具类）