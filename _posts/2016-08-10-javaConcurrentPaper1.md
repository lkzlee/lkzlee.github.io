---
layout: post
title: Java并发编程艺术-并发编程的挑战（第一章）
date:   2016-08-10 21:26:00
categories: java并发编程
tags: 技术积累
---
*****
* TOC
{:toc}
*****

# 1.1 并发编程

并发编程的本质：让程序执行的更加快速。并不是启动更多的线程让程序最大限度并发执行。

# 1.2 上下文切换

时间片：CPU分配给各个线程的执行时间。

上下文切换：CPU通过时间片分配算法来循环执行程序任务，当前执行的任务执行一个时间片后会切换到下一个任务，但是切换前必须保持上一个任务的状态，以便保证下次切换回任务可以从上次执行处继续执行。

很明显，线程执行的上下文切换会影响多线程的执行速度。

# 1.3 并发编程验证
代码示例，看看是否并发就比串行执行的快：

~~~java
package com.lkzlee.example;

public class ConcurrencyTest
{
	private static long count = 100000l;

	public static void main(String[] args) throws InterruptedException
	{
		concurrencyTest();
		serialTest();
	}
	private static void serialTest()
	{
		long startTime = System.currentTimeMillis();
		int a = 0;
		for (long i = 0; i < count; i++)
		{
			a += 5;
		}
		int b = 0;
		for (long i = 0; i < count; i++)
		{
			b--;
		}
		System.out.println("serial cost:" + (System.currentTimeMillis() - startTime) + " ms");
	}
	private static void concurrencyTest() throws InterruptedException
	{
		long startTime = System.currentTimeMillis();
		Thread t = new Thread(new Runnable() {
			@Override
			public void run()
			{
				int a = 0;
				for (long i = 0; i < count; i++)
				{
					a += 5;
				}
			}
		});
		t.start();
		int b = 0;
		for (long i = 0; i < count; i++)
		{
			b--;
		}
		t.join();
		System.out.println("concurrency cost:" + (System.currentTimeMillis() - startTime) + " ms");
	}
}

~~~

自己按照书本上了做了下测试，每次表现不太一样，但是确实可以验证有时候并发的执行时间不一定比串行程序执行时间短，有可能会出现比串行时间长。

# 1.4 上下文切换次数查询命令

上线文切换的次数可以通过`vmstat`来查看，cs（content switch）表示上下文切换的次数。

~~~shell
 vmstat 1
procs -----------memory---------- ---swap-- -----io---- -system-- ------cpu-----
 r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
 1  0      0 189600      0 2987724    0    0     0     3    1    1  0  0 99  0  0
 0  0      0 189584      0 2987740    0    0     0     0  778 1573  0  0 99  0  0
 0  0      0 189584      0 2987756    0    0     0     0  745 1524  0  0 99  0  0
 0  0      0 189460      0 2987772    0    0     0     8 1006 1778  1  0 99  0  0
 0  0      0 189584      0 2987680    0    0     0     0  878 1640  0  0 100  0  0
 2  0      0 189584      0 2987692    0    0     0     0  910 1637  1  0 99  0  0
 0  0      0 189584      0 2987712    0    0     0     0  793 1610  1  0 99  0  0

~~~

# 1.5 如何减少上下文切换

减少上下文切换的方式：无锁并发编程、CAS算法、使用最小线程和使用协程。

无锁并发编程：多线程竞争锁时，会引起上下文切换，所以多线程处理数据时，可以采用一个办法避免使用锁，如将数据的Id%段进行hash分段，不同线程处理不同的段。

CAS算法：Java中CAS都是Atomic原子操作，可以不用加锁进行多线程处理。
> CAS通过调用JNI的代码实现的。JNI:Java Native Interface为JAVA本地调用，允许java调用其他语言。
而compareAndSwapInt就是借助C来调用CPU底层指令实现的。

使用最小线程：越少的线程，可以避免上下文的切换的时间消耗代价，提高程序执行效率。避免不需要的线程。

协程：在单线程中实现多任务调度，并在单线程中维持多个任务间的切换。


# 1.6 死锁

死锁发生的本质，引用操作系统的一句话，相互持并等待的场景下发生死锁。

死锁代码示例：

~~~java
package com.lkzlee.example;

public class DeadLockDemo
{
	private static String lockA = "LOCKA";
	private static String lockB = "LOCKB";

	public static void main(String[] args) throws InterruptedException
	{
		new DeadLockDemo().deadLockTest();
	}

	private void deadLockTest() throws InterruptedException
	{
		Thread t1 = new Thread(new Runnable() {

			@Override
			public void run()
			{
				synchronized (lockA)
				{
					synchronized (lockB)
					{
						try
						{
							Thread.sleep(1000);
							System.out.println("thead1 get lockA lockB..... ");
						}
						catch (InterruptedException e)
						{
							e.printStackTrace();
						}
					}
				}

			}
		});
		Thread t2 = new Thread(new Runnable() {

			@Override
			public void run()
			{
				synchronized (lockB)
				{

					synchronized (lockA)
					{
						try
						{
							Thread.sleep(1000);
							System.out.println("thead2 get lockB lockA..... ");
						}
						catch (InterruptedException e)
						{
							// TODO Auto-generated catch block
							e.printStackTrace();
						}
					}
				}

			}
		});
		t1.start();
		t2.start();
		t1.join();
		t2.join();
		System.out.println("main Thread execute end....");
	}
}

~~~

改死锁示例并不是一定会死锁，而是很可能发生死锁这一点需要注意，在示例代码中thread1获取lockA，等待lockB，thread2获取lockB，等待lockA，这样就会放手相互持有各自一个资源并等待另外线程持有的资源，此时死锁就发生了。

死锁的常用避免方法（java代码开发中，和操作系统的中的描述不是一个）：

1. 避免一个线程持有多个锁资源
2. 避免一个线程在锁内同时占有多个资源，尽量保证只占用一个资源。
3. 尝试使用lock.tryLock()这样获取不到不用阻塞，可以立即返回
4. 数据库锁的，加锁和解锁必须在一个数据库连接中，一般spring 从代码级已经保证，还有一个点，减少锁持有的时间也可以很大程度的提交程序执行效率和数据库连接的利用率。