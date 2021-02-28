---
layout: post
title: 批量删除Redis数据库中的Key
date: 2016-06-16
tags:
- Redis
- 转载
categories: 分布式系统
description: 批量删除Redis数据库中的Key
---
> 转载自：http://img.snail8.com/?p=502

Redis中有删除单个Key的指令`DEL`，但好像没有批量删除Key的指令，不过我们可以借助Linux的`xargs`指令来完成这个动作。

```bash
redis-cli keys "*" | xargs redis-cli del
# 如果redis-cli没有设置成系统变量，需要指定redis-cli的完整路径
# 如：/opt/redis/redis-cli keys "*" | xargs /opt/redis/redis-cli del
```

如果要指定 Redis 数据库访问密码，使用下面的命令
```bash
redis-cli -a password keys "*" | xargs redis-cli -a password del
```

如果要访问 Redis 中特定的数据库，使用下面的命令
```bash
# 下面的命令指定数据序号为0，即默认数据库
redis-cli -n 0 keys "*" | xargs redis-cli -n 0 del
```

删除所有Key，可以使用Redis的flushdb和flushall命令
```bash
# 删除当前数据库中的所有Key
flushdb
# 删除所有数据库中的key
flushall
```
