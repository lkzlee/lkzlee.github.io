---
layout: post
title: JVM及tomcat启动配置相关参数
date:   2016-12-06 11:37:47
categories: java技术
tags: 技术积累
---
*****
* TOC
{:toc}
*****


# tomcat启动参数

在公司经常看到jvm参数调优等一系列参数，于是乎看了这一份启动参数，对这些参数产生了兴趣，这些究竟能对tomcat做什么，使得jvm获得更好的性能。下面是一份启动样例：
~~~shell
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
~~~

# jvm启动参数调优

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
- -XX:+PrintGCDetails  打印GC详细的信息，如：`2016-12-06T13:34:25.784+0800: 12208.859: [GC (Allocation Failure) 2016-12-06T13:34:25.784+0800: 12208.859: [ParNew: 316329K->1662K(353920K), 0.0139227 secs] 663846K->349184K(1009280K), 0.0141961 secs] [Times: user=0.04 sys=0.00, real=0.02 secs]` 
- -verbose:gc 
- -Xloggc:/home/xxx/logs/gc.log  gc输出日志文件
- 
# tomcat按项目启动