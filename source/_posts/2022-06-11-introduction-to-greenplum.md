---
layout: post
title: GreenPlum入门
date: 2022-06-11
tags:
- GreenPlum
categories: BigData
description: 本文旨在小白入门GreenPlum。
---

## GPDB简介

> Pivotal Greenplum Database is a MPP (massively parallel processing) database built on open source [PostgreSQL](https://en.wikipedia.org/wiki/PostgreSQL?spm=a2c6h.12873639.article-detail.4.1bcc51349E2X2H).The system consists of a master node, standby master node, and segment nodes.

> All of the data resides on the segment nodes and the catalog information is stored in the master nodes. Segment nodes run one or more segments, which are modified PostgreSQL database instances and are assigned a content identifier.

> For each table the data is divided among the segment nodes based on the distribution column keys specified by the user in the DDL statement.

> For each segment content identifier there is both a primary segment and mirror segment which are not running on the same physical host.

> When a SQL query enters the master node, it is parsed, optimized and dispatched to all of the segments to execute the query plan and either return the requested data or insert the result of the query into a database table.

从定义上看，GP和此前我比较专注的ES(Elasticsearch)非常类似，它们都是基于成熟的单机技术——PG(PostgreSQL)和Lucene，采用去中心化的分布式技术，实现了对大数据的(MPP和信息检索)支持。

## 安装GreenPlum

一下安装针对Ubuntu系统。

### 添加镜像源

如果是Ubuntu 16.04或Ubuntu 18.04，执行以下命令安装：

```bash
sudo add-apt-repository ppa:greenplum/db
```

如果是Ubuntu 20.04或其他，需要手动添加`source.list`，创建文件`/etc/apt/sources.list.d/greenplum-ubuntu-db-focal.list`：
```
deb https://ppa.launchpadcontent.net/greenplum/db/ubuntu bionic main
deb-src https://ppa.launchpadcontent.net/greenplum/db/ubuntu bionic main
```

### 安装GreenPlum

执行以下命令安装：

```bash
sudo apt update
sudo apt install greenplum-db-6
```

The above command will install the Greenplum Database software and any required dependencies on the system automatically and put the resulting software in /opt directory as seen below:

```

```

接下来使用如下命令下载GP官方提供的教程示例，然后解压到任意目录：
