---
layout: post
title: Redis 使用中遇到的相关问题
date:   2016-08-10 21:26:00
categories: Redis技术
tags: 技术积累
---
1.1 Redis使用过程中的问题
问题1，redis发布订阅过程中，使用该连接进行其他处理，导致订阅失败。错误栈如下：

~~~console
redis.clients.jedis.exceptions.JedisDataException: ERR only (P)SUBSCRIBE / (P)UNSUBSCRIBE / QUIT allowed in this context
        at redis.clients.jedis.Protocol.processError(Protocol.java:66)
        at redis.clients.jedis.Protocol.process(Protocol.java:73)
        at redis.clients.jedis.Protocol.read(Protocol.java:138)
        at redis.clients.jedis.Connection.getBinaryBulkReply(Connection.java:185)
        at redis.clients.jedis.Connection.getBulkReply(Connection.java:174)
        at redis.clients.jedis.Jedis.get(Jedis.java:77)
~~~

我们公司实现Redis发布订阅，实现方式和以下代码类似，使用一个Redis订阅的类继承至JedisPubSub，然后定义一个`JedisSubscribeHandler`接口,该接口的实现类需要调用`RedisSubListener`的注册方法`registerHandler`，这样每次redis发布一条消息就可以通过channle分发到具体的handler做处理。

~~~java

import org.apache.commons.logging.Log;
import org.apache.commons.logging.LogFactory;

import redis.clients.jedis.JedisPubSub;

import com.lkzlee.redis.handler.JedisSubscribeHandler;

/**
 * 继承自JedisPubSub实现redis的发布订阅
 * @author lkzlee
 *
 */
public class RedisSubListener extends JedisPubSub
{
	private final static Map<String, JedisSubscribeHandler> registerHandlerMap = new HashMap<String, JedisSubscribeHandler>();
	private final static Log log = LogFactory.getLog(RedisSubListener.class);

	public static void registerHandler(String channel, JedisSubscribeHandler handler)
	{
		registerHandlerMap.put(channel, handler);
	}

	@Override
	public void onMessage(String channel, String message)
	{
		if (registerHandlerMap.containsKey(channel))
		{
			log.fatal("找不到相关channel订阅中信息，channel=" + channel + "|message=" + message + ",请检查是否有注册");
			return;
		}
		JedisSubscribeHandler handler = registerHandlerMap.get(channel);
		handler.handler(message);
	}
}
~~~
我们内部对于获取Jedis连接做了一些优化，实现了`JedisProxy`类,这个类可以防止同步调用方法类获取多个Jedis连接，已提高redis利用率，并通过一个JedisProxyAop注入的方法不用再执行Jedis方法的同时获取和释放连接，具体实现原理如下：
JedisProxy内部有一个成员：
~~~java

	/**
	 * 获取jedis
	 * @return
	 */
	public Jedis getJedis()
	{
		Jedis jedis = null;
		try
		{
			/**去本线程中的jedis*/
			jedis = jedisLocal.get();
			if (null != jedis)
			{
				try
				{
					// 取出来后执行ping检查下是否依然存活
					if (jedis.isConnected())
					{
						return jedis;
					}
				}
				catch (Exception e)
				{
					releaseBrokenJedis();
				}
			}
			if (null == redisSentinelPool)
			{
				this.initPool();
			}
			jedis = redisSentinelPool.getResource();
			jedisLocal.set(jedis);// 设置本线程中的jedis
		}
		catch (Exception e)
		{
			releaseBrokenJedis();
			if (null != jedisLocal.get())
			{
				jedisLocal.remove();
			}
			log.fatal("Could not get a resource from the pool, pls check the host and port settings", e);
		}
		return jedis;
	}
	/**
	 * 释放
	 */
	public void releaseJedis()
	{
		Jedis jedis = jedisLocal.get();
		if (null != jedis && null != redisSentinelPool)
		{
			if (jedis.isConnected())
			{
				redisSentinelPool.returnResource(jedis);
			}
		}
		jedisLocal.remove();
	}

	/**
	 * 释放损坏资源
	 */
	public void releaseBrokenJedis()
	{
		Jedis jedis = jedisLocal.get();
		if (null != jedis && null != redisSentinelPool)
		{
			redisSentinelPool.returnBrokenResource(jedis);
		}
		jedisLocal.remove();
	}
~~~


`redisSentinelPool`这个类基于`JedisSentinelPool`实现了Jedis连接池管理，但是各位有没有发现`handler.handler(message);`调用的这个方法是同步调用的，如果此方法处理的业务逻辑较多，比较又用到了redis，此时获取的redis连接和`JedisPubSub`【可以看看源码，有内置的client】连接是同一个的话，就会造成Redis上述的报错。


问题2：

Jedis返回`JedisConnectionException:Unknown reply: /` 类似的错误，其本质可以通过源码看到Redis获取连接和使用是通过Pool来进行管理的，当时获取一个redis连接的时候`internalPool.borrowObject()`从Jedis实现的连接池中获取一个，用完释放`returnResourceObject(resource)`还给连接池，如果有一次调用Jedis链接获取缓存内容，返回过程中异常，而改连接流中的数据未处理为清空，那下次另一个线程获取到连接请求获取的消息时，上次信息会继续返回，此时返现连接请求的返回数据不一致就会报错。


如下代码：

~~~java
   private static Object process(final RedisInputStream is) {
	try {
	    byte b = is.readByte();
	    if (b == MINUS_BYTE) {
		processError(is);
	    } else if (b == ASTERISK_BYTE) {
		return processMultiBulkReply(is);
	    } else if (b == COLON_BYTE) {
		return processInteger(is);
	    } else if (b == DOLLAR_BYTE) {
		return processBulkReply(is);
	    } else if (b == PLUS_BYTE) {
		return processStatusCodeReply(is);
	    } else {
		throw new JedisConnectionException("Unknown reply: " + (char) b);
	    }
	} catch (IOException e) {
	    throw new JedisConnectionException(e);
	}
	return null;
    }

~~~