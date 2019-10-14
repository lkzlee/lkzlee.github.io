# Cassandra问题分析及总结

这边文章对Cassandra的此处安装配置涉及不多，只给出了相关链接，主要列举了核心参数调优和问题记录。

## 存在问题

Cassandra目前存在性能瓶颈，写性能上不去，在写入性能达18MB/s时已经达到瓶颈，速度上不去。

报错：com.datastax.driver.core.exceptions.WriteTimeoutException:Cassandra timeout during write query at consistency LOCAL_ONE(1 replica were required but only 0 acknowledged the write)

## 目标

优化Cassandra，使得Cassandra写性能达到GB级别/s，排查出问题。

## 问题检索

相关链接：

性能优化及常用排查手段 https://blog.csdn.net/wangtua/article/details/78358639

Cassandra简介及下载地址  http://cassandra.apache.org/ Manage massive amounts of data, fast, without losing sleep

Cassandra安装配置及调试：https://www.cnblogs.com/zlslch/p/8031548.html (一开始没有虚拟机资源，搭建到本地windows了)

Cassandra配置disk_access使用内存映射方式mmap可能产生堆外内存问题 http://baijiahao.baidu.com/s?id=1642281319415881720&wfr=spider&for=pc 



comitlog相关 https://blog.csdn.net/albertfly/article/details/51542154

## Cassandra配置确认

num_tokens 定义了随机分配给换上该节点的令牌数，相对于其他节点，令牌数越多该节点存储的数据比例越大，如果各节点机器具备相同的硬件能力，他们应该具备相同的令牌数。

batchlog_replay_throttle_in_kb 文件batchlog写入速率，单位KB/s，默认1024，这个速度按节点数比例调整即可。

hints_flush_period_in_ms 刷盘的时间周期频率，适当调整

concurrent_reads  并发读的数量
concurrent_writes  并发写的数量
concurrent_counter_writes 并发写计数器值

这里文档中有个公式：

​	并发读 =16 * number_of_drives（我理解的是整个机器对外提供全部读的qps），这个值的合理设置我觉得可以提高读效率， 降低操作系统中各io读请求的响应速度和吞吐量。

​	并发写= 8 * number_of_cores （cpu核心数），Cassandra写入几乎没有IO限制，我觉得根据自己条件可以调高试试，找到适合自己的经验值最好。

​	concurrent_counter_writes  这个参数是写入的最大阈值，计数器写入递增和写回它们之前读取当前值。

​	这三个参数concurrent_reads 、concurrent_writes、concurrent_counter_writes  根据自身业务调整合理的经验值，应该能够获得良好的收益。

disk_optimization_strategy 这个是系统配置的磁盘策略，根据硬盘的不同可调整，ssd最好用ssd策略，否则系统默认会走spinning磁盘扫描算法也就是现有磁盘的扫描算法（电梯算法，FIFO，最短询道时间，分步电梯算法等等）。

memtable_allocation_type 内存管理的方式，这个参数我觉得也很重要，根据我们的业务场景，读特别频繁，还是写频繁可以做一下研究，修改内存管理和分配方式。



commitlog_sync commitlog同步方式：定期或者batch批量两种策略。

## 可进行的尝试和排查手段

> 进行Cassandra参数配置调优，用二分法尝试最优值，可参加上面列的Cassandra配置
>
> Cassadra本身基于java，也可以分析jvm内存分析，做jvm相关参数调优，还有第一篇文章中提到的线程压缩优化。
>
> 操作系统硬件内存，io硬件参数调整，更好满足cassandra性能提升。
>
> 查看commitlog 大小，查看系统log，当时写入操作记录都是那些，针对业务进行分析。查看对于的表排查索引问题：是否有索引或查询能用到的索引。



## 存在的问题

业务信息不是很充足，最好能够有权限进入Cassandra服务，然后从系统各个维度进行分析，更加深入地理解业务场景，继续检索和排查对应的问题。



## 遇到的问题record



1 网络速度跟不上，比较耽误时间，安装Cassandra依赖的jdk、python比较费时间

2 启动后，按照客户端依赖csqlsh，发现需要安装Cpython，thrift--》安装Cpython需要pip--》安装pip需要安装setuptool，由于网络问题，这条依赖链下载各个基础库经常失败，各种尝试终于安装成功。

3 由于安装pip后下载会走国外源，修改pip源改为国内清华源提高下载安装速度

4 使用命令进行连接，使用`chcp 65001`命令window dos支持utf-8，`cqlsh localhost 9042` 连接客户端，进行测试，创建keyspace，table，insert数据，进行查询分析，使用`desc [tablename]`可查看表结构信息。