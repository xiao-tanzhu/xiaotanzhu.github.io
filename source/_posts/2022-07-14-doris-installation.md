---
layout: post
title: 编译和安装Doris
date: 2022-07-14
tags:
- 数据仓库
- Doris
categories: 数据仓库
description: 本文档主要介绍如何通过源码编译 Doris，并安装Doris到服务器。
---

## 编译Doris

### 使用 Docker 开发镜像编译（推荐）

#### 使用现成的镜像

下载镜像：
```bash
docker pull apache/incubator-doris:build-env-for-1.0.0
```

下载源码到本地（目录：/home/ubuntu/doris）：
```
wget https://dist.apache.org/repos/dist/release/doris/1.0/1.0.0-incubating/apache-doris-1.0.0-incubating-src.tar.gz
```

运行编译环境：
```
docker run -it -v /home/ubuntu/.m2:/root/.m2 -v /home/ubuntu/doris/apache-doris-1.0.0-incubating-src:/root/apache-doris-1.0.0-incubating-src apache/incubator-doris:build-env-for-1.0.0
```

## 修改配置文件

修改fe.conf和be.conf的网络配置，增加以下行：
```
priority_networks=10.1.3.0/24
```

## 启动程序

### 启动FE
```bash
bin/start_fe.sh --daemon
```

### 增加BE
```sql
ALTER SYSTEM ADD BACKEND "be_host:heartbeat-service_port";
```

其中`heartbeat-service_port`默认为：9050

### 启动BE

```bash
bin/start_broker.sh --daemon
```
