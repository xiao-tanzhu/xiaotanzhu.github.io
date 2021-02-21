---
layout: post
title: Redis提示Could not get a resource from the pool（jedis连接池配置）
tags:
- Redis
- Jedis
- 转载
categories: 分布式系统
description: Redis提示Could not get a resource from the pool（jedis连接池配置）
---

起初在JedisPool中配置了50个活动连接，但是程序还是经常报错：Could not get a resource from the pool

连接池刚开始是这样配置的：
```java
JedisPoolConfig config = new JedisPoolConfig();
config.setMaxTotal(50);
config.setMaxIdle(20); 
config.setMaxWaitMillis(1000 * 1);
config.setTestOnBorrow(true);
config.setTestOnReturn(true);
JedisPool pool = new JedisPool(config, "10.10.10.167", 6379);
```
经过测试发现程序的活动连接基本上只有1个，程序刚启动的时候可能会有2-5个活动的连接，但是过一段时间后就获取不到第二个活动的连接了。

后来修改为：
```java
JedisPoolConfig config = new JedisPoolConfig();
config.setMaxTotal(200); 
config.setMaxIdle(50);
config.setMinIdle(8);//设置最小空闲数 
config.setMaxWaitMillis(10000);
config.setTestOnBorrow(true);
config.setTestOnReturn(true); //Idle时进行连接扫描 
config.setTestWhileIdle(true); //表示idle object evitor两次扫描之间要sleep的毫秒数
config.setTimeBetweenEvictionRunsMillis(30000); //表示idle object evitor每次扫描的最多的对象数
config.setNumTestsPerEvictionRun(10); //表示一个对象至少停留在idle状态的最短时间，然后才能被idle object evitor扫描并驱逐；这一项只有在timeBetweenEvictionRunsMillis大于0时才有意义 
config.setMinEvictableIdleTimeMillis(60000);
JedisPool pool = new JedisPool(config, ip, port, 10000, "密码", 0);
```

经过几个小时的测试，读取redis了上百万次再也没有发生上述错误。

在这里进行简单的猜测：连接池中空闲的连接过一阵子就会自动断开，但是连接池还以为连接正常，就出现了这个错误。

另外，从连接池中获取连接的时候，可以写个循环，直到获取成功才让出循环。