---
layout: post
title: 创建支持emoji表情的MySQL数据库（utf8mb4）
tags:
- MySQL
- emoji
- utf8mb4
categories: 数据库
description: 创建支持emoji表情的MySQL数据库（utf8mb4）
---

>本文只介绍在创建全新数据库的情况下，如何支持emoji表情等字符。如果需要对现有的数据库修改以支持emoji表情，请参考：[How to support full Unicode in MySQL databases](/2016/08/14/how-to-support-full-unicode-in-mysql.html)

UTF-8编码有可能是两个、三个、四个字节。Emoji表情是4个字节，而MySQL的utf8编码最多3个字节，当使用iPhone等插入表情的时候，会抛出如下错误：
```
java.sql.SQLException: Incorrect string value: '\xF0\x9F\x92\x94' for column 'name' at row 1
    at com.mysql.jdbc.SQLError.createSQLException(SQLError.java:1073)
    at com.mysql.jdbc.MysqlIO.checkErrorPacket(MysqlIO.java:3593)
    at com.mysql.jdbc.MysqlIO.checkErrorPacket(MysqlIO.java:3525)
    at com.mysql.jdbc.MysqlIO.sendCommand(MysqlIO.java:1986)
    at com.mysql.jdbc.MysqlIO.sqlQueryDirect(MysqlIO.java:2140)
    at com.mysql.jdbc.ConnectionImpl.execSQL(ConnectionImpl.java:2620)
    at com.mysql.jdbc.StatementImpl.executeUpdate(StatementImpl.java:1662)
    at com.mysql.jdbc.StatementImpl.executeUpdate(StatementImpl.java:1581)
```
解决方案就是：**将Mysql的编码从`utf8`转换成`utf8mb4`**。

## MySQL服务器配置

修改MySQL配置文件`/etc/mysql/my.cnf`：
```
[mysqld]
character-set-server=utf8mb4
[mysql]
default-character-set=utf8mb4
```
这样，在创建数据库或数据库表的时候，如果不指定编码格式，则默认使用`utf8mb4`。

## 创建库的时候指定

```mysql
CREATE DATABASE `irenshi` DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci
```

## 使用Docker时

如果你使用MySQL的[官方Docker](https://hub.docker.com/_/mysql/)启动并创建数据库，官方文档也给出了启动命令：
```bash
docker run --name some-mysql -e MYSQL_ROOT_PASSWORD=my-secret-pw -d mysql:tag --character-set-server=utf8mb4 --collation-server=utf8mb4_unicode_ci
```
