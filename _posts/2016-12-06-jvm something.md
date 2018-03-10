---
layout: post
title: JVM配置相关参数
date:   2016-12-06 11:37:47
categories: java技术
tags: 技术积累
description: 一篇jvm参数配置详解清单
---
*****
* TOC
{:toc}
*****


# 一份tomcat启动参数的样例

在公司经常看到jvm参数调优等一系列参数，于是乎看了这一份启动参数，对这些参数产生了兴趣，这些究竟能对tomcat做什么，使得jvm获得更好的性能。下面是一份启动样例：
<pre>
/home/jdk1.8.0/bin/java
 -Djava.util.logging.config.file=/home/xxx/conf/logging.properties 
 -Djava.util.logging.manager=org.apache.juli.ClassLoaderLogManager 
 -Djdk.tls.ephemeralDHKeySize=2048 
 -Xmn384m 
 -Xms1024m 
 -Xmx1024m 
 -XX:PermSize=128m 
 -XX:MaxPermSize=256m 
 -XX:+UseParNewGC 
 -XX:+UseConcMarkSweepGC 
 -XX:CMSFullGCsBeforeCompaction=0 
 -XX:+CMSClassUnloadingEnabled 
 -XX:+CMSParallelRemarkEnabled 
 -XX:+CMSScavengeBeforeRemark 
 -XX:CMSInitiatingOccupancyFraction=60 
 -XX:+UseCMSCompactAtFullCollection 
 -XX:+DisableExplicitGC 
 -XX:+PrintGCTimeStamps 
 -XX:+PrintGCDateStamps 
 -XX:+PrintGCDetails 
 -verbose:gc 
 -Xdebug 
 -Xloggc:/home/xxx/logs/gc.log 
 -Djava.endorsed.dirs=/home/tomcat8/endorsed 
 -classpath /home/tomcat7/bin/bootstrap.jar:/home/tomcat7/bin/tomcat-juli.jar 
 -Dcatalina.base=/home/xxx
 -Dcatalina.home=/home/tomcat8 
 -Djava.io.tmpdir=/home/project/xxx/temp org.apache.catalina.startup.Bootstrap start
</pre>

# JVM参数粗解

## jvm各个参数意义

- -xmn 年轻代大小。注意此处的大小是（eden+ 2 survivor space).与jmap -heap中显示的New gen是不同的。
整个堆大小=年轻代大小 + 年老代大小 + 持久代大小.
增大年轻代后,将会减小年老代大小.此值对系统性能影响较大,Sun官方推荐配置为整个堆的3/8

- -xms 初始堆大小。

- -Xmx1024m 最大堆大小。 

- -Djava.util.logging.config.file java日志使用的配置

- -Djava.util.logging.manager  java日志管理使用tomcat中的ClassLoaderLogManager
 
- -XX:PermSize 设置持久代(perm gen)初始值
 
- -XX:MaxPermSize 	设置持久代最大值

- -XX:+UseParNewGC 	设置年轻代为并行收集
 
- -XX:+UseConcMarkSweepGC 	使用CMS内存收集器
 
- -XX:CMSFullGCsBeforeCompaction=0 	多少次后进行内存压缩。由于并发收集器不对内存空间进行压缩,整理,所以运行一段时间以后会产生"碎片",使得运行效率降低.此值设置运行多少次GC以后对内存空间进行压缩,整理.

- -XX:+CMSClassUnloadingEnabled  并发对PermGen进行收集，只有对于cms垃圾回收器有用，对于G1回收器则只能通过fullgc

- -XX:+CMSParallelRemarkEnabled  降低标记停顿，使用并行标记

- -XX:+CMSScavengeBeforeRemark 

- -XX:CMSInitiatingOccupancyFraction=60  使用CMS作为垃圾回收，使用60％后开始CMS收集

- -XX:+UseCMSCompactAtFullCollection  在FULL GC的时候，对年老代的压缩

- -XX:+DisableExplicitGC 关闭System.gc()

- -XX:+PrintGCTimeStamps 打印GC时间戳

- -XX:+PrintGCDateStamps 

- -XX:+PrintGCDetails  打印GC详细的信息，如：2016-12-06T13:34:25.784+0800: 12208.859: [GC (Allocation Failure) 2016-12-06T13:34:25.784+0800: 12208.859: [ParNew: 316329K->1662K(353920K), 0.0139227 secs] 663846K->349184K(1009280K), 0.0141961 secs] [Times: user=0.04 sys=0.00, real=0.02 secs] 

- -verbose:gc 表示输出虚拟机中GC的详细情况
 
- -Xloggc:/home/xxx/logs/gc.log  gc输出日志文件

网上还有很多类似的参数，jvm调优的本质是根据自己的业务场景合理的分配和回收内存，每个产品的业务可能不一致，因此根据产品业务设置合理的jvm参数是必要的, 合理的内存分配与回收可以很高的提高应用的性能。

网上关于参数的描述还有有很多，以下来自[http://www.mamicode.com/info-detail-1028149.html](http://www.mamicode.com/info-detail-1028149.html)

## java JVM的垃圾回收器选择：

- -XX:+UseSerialGC:设置串行收集器

- -XX:+UseParallelGC:设置并行收集器

- -XX:+UseParalledlOldGC:设置并行年老代收集器

- -XX:+UseConcMarkSweepGC:设置并发收集器

## 垃圾回收统计信息

- -XX:+PrintGC

- -XX:+PrintGCDetails

- -XX:+PrintGCTimeStamps

- -Xloggc:filename

## 并行收集器设置

- -XX:ParallelGCThreads=n:设置并行收集器收集时使用的CPU数。并行收集线程数。

- -XX:MaxGCPauseMillis=n:设置并行收集最大暂停时间

- -XX:GCTimeRatio=n:设置垃圾回收时间占程序运行时间的百分比。公式为1/(1+n)

## 并发收集器设置

- -XX:+CMSIncrementalMode:设置为增量模式。适用于单CPU情况。

- -XX:ParallelGCThreads=n:设置并发收集器年轻代收集方式为并行收集时，使用的CPU数。并行收集线程数。

## 堆设置

- -Xms:初始堆大小

- -Xmx:最大堆大小

- -XX:NewSize=n:设置年轻代大小

- -XX:NewRatio=n:设置年轻代和年老代的比值。如:为3，表示年轻代与年老代比值为1：3，年轻代占整个年轻代年老代和的1/4

- -XX:SurvivorRatio=n:年轻代中Eden区与两个Survivor区的比值。注意Survivor区有两个。如：3，表示Eden：Survivor=3：2，一个Survivor区占整个年轻代的1/5

- -XX:MaxPermSize=n:设置持久代大小
