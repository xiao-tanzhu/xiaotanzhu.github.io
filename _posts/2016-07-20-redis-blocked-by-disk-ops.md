---
layout: post
title: Redis被bgsave和bgrewriteaof阻塞的解决方法
tags:
- Redis
- Performance
categories: 分布式系统
description: Redis被bgsave和bgrewriteaof阻塞的解决方法
---
我们的系统由于业务特点在每天早上上班的时候，会出现访问高峰。最近发现在访问高峰的时候，系统响应会变慢。无理是APP的后端还是Web服务的后端，平时响应时间都在100ms以内的接口，响应时间居然超过了一秒，甚至有一些能够达到10秒。

查看后台的监控数据，无论是服务器硬件资源利用率还是数据库访问频次都在正常的范围内。那么问题非常可能出现在Redis上，因为在我们的系统中，Redis担任了所有Web端和APP端Session的管理，权限的校验以及数据库的缓存等等任务。

查看Redis的日志，果然发现了异常：后台不停的打出bgsave相关的日志。

###bgsave频率的问题
查看`/etc/redis/redis.conf`配置，发现了下面这一段：
```
################################ SNAPSHOTTING  ################################
#
# Save the DB on disk:
#
#   save <seconds> <changes>
#
#   Will save the DB if both the given number of seconds and the given
#   number of write operations against the DB occurred.
#
#   In the example below the behaviour will be to save:
#   after 900 sec (15 min) if at least 1 key changed
#   after 300 sec (5 min) if at least 10 keys changed
#   after 60 sec if at least 10000 keys changed
#
#   Note: you can disable saving at all commenting all the "save" lines.
#
#   It is also possible to remove all the previously configured save
#   points by adding a save directive with a single empty string argument
#   like in the following example:
#
#   save ""

save 900 1
save 300 10
save 60 10000
```
这三行的意思是：如果Redis在60秒之内有超过10000次修改，那么触发一次磁盘快照；或者在300秒内如果有10次修改，则触发一次快照；或者在900秒内有一次修改，则触发磁盘快照。

在我们的系统中，早晨访问高峰时，60秒内Redis的访问次数一定会超过10000次，所以系统就在不停的进行磁盘快照，于是就有了bgsave相关的日志。

###bgsave影响系统性能？
问题又来了。按字面理解，`bgrewriteaof`是在后台进行操作，不应该影响Redis的正常服务。原理也确实是这样的，Redis首先fork一个子进程，并在该子进程里进行归并和写持久化存储设备（如硬盘）的。按照正常逻辑，在一台多核的机器上，即使子进程占满CPU和硬盘,也不应该导致Redis服务阻塞啊！

Google了一下，发现问题就出在硬盘上。

Redis服务设置了`appendfsync everysec`，主进程每秒钟便会调用`fsync()`，要求内核将数据”确实”写到存储硬件里。但由于子进程同时也在写硬盘，从而导致主进程`fsync()/write()`操作被阻塞，最终导致Redis主进程阻塞了。

###解决方法
解决方法便是设置
```
no-appendfsync-on-rewrite yes
```
在子进程处理和写硬盘时，主进程不调用 fsync() 操作。需要注意的是，即使进程不调用`fsync()`，系统内核也会根据自己的算法在适当的时机将数据写到硬盘（Linux默认最长不超过30秒）。

###进一步提速：降低磁盘快照频率
由于目前我们的系统中，Redis真的是做为缓存，并且只作为缓存，不处理任何持久性数据，所以不需要快照的如此频繁。于是修改了快照的频率，把以下内容
```
save 900 1
save 300 10
save 60 10000
```
修改为：
```
save 900 1
```
即，900秒内，如果有一次修改，则进行一次快照。于是乎，问题搞定！


