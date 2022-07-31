---
layout: post
title: Doris常用命令
date: 2022-07-31
tags:
- 数据仓库
- Doris
categories: 数据仓库
description: Doris常用命令
---

## 连接Doris

```bash
mysql -h doris-fe -P 9030 -uroot
```

默认root密码为空。

## 节点管理

### 增加Backend

```sql
ALTER SYSTEM ADD BACKEND "be_host:heartbeat-service_port";
```

`heartbeat-service_port`默认为9050，即：

```sql
ALTER SYSTEM ADD BACKEND "be_host:9050";
```

### 查看Backend
```
SHOW PROC '/backends'
```

### 增加Broker

```sql
ALTER SYSTEM ADD BROKER broker_name "broker_host1:broker_ipc_port1","broker_host2:broker_ipc_port2",...;
```

## 启动服务

### 启动FE

```bash
bin/start_fe.sh --daemon
```

### 启动BE

```bash
bin/start_be.sh --daemon
```

### 启动Broker

```bash
bin/start_broker.sh --daemon
```
