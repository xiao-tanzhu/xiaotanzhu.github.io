---
layout: post
title: 开始使用Mesos和Marathon
tags:
- Mesos
- Marathon
categories: 运维
description: 开始使用Mesos和Marathon
---

Marathon 是可以跟 Mesos 一起协作的一个 framework，用来运行持久性的应用。

## 安装

一共需要安装四种组件，mesos-master、marathon、zookeeper 需要安装到所有的主节点，mesos-slave 需要安装到从节点。mesos 利用 zookeeper 来进行主节点的同步，以及从节点发现主节点的过程。

### 获取Docker镜像

在主节点上拉取mesos-master、marathon、zookeeper相关镜像：
```bash
docker pull garland/zookeeper
docker pull garland/mesosphere-docker-mesos-master
docker pull garland/mesosphere-docker-marathon
```
在从节点上拉取mesos-slave镜像，其实它和mesos-master是同一个镜像，只是启动参数不同：
```bash
docker pull garland/mesosphere-docker-mesos-master
```

### 启动主节点相关服务

**启动zookeeper**

```bash
docker run -d -p 2181:2181 -p 2888:2888 -p 3888:3888 garland/zookeeper
```

**启动mesos-master**

```bash
export HOST_IP=192.168.1.2
docker run --net="host" --name mesos_master -p 5050:5050 \
-e "MESOS_HOSTNAME=${HOST_IP}" \
-e "MESOS_IP=${HOST_IP}" \
-e "MESOS_ZK=zk://${HOST_IP}:2181/mesos" \
-e "MESOS_PORT=5050" \
-e "MESOS_LOG_DIR=/var/log/mesos" \
-e "MESOS_QUORUM=1" \
-e "MESOS_REGISTRY=in_memory" \
-e "MESOS_WORK_DIR=/var/lib/mesos" \
-d garland/mesosphere-docker-mesos-master
```

**启动marathon**

```bash
export HOST_IP=192.168.1.2
docker run -d --name mesos_marathon -p 8080:8080 --master zk://${HOST_IP}:2181/mesos --zk zk://${HOST_IP}:2181/marathon garland/mesosphere-docker-marathon
```

### 启动从节点相关服务

```bash
export HOST_IP=192.168.1.2
docker run -d --name mesos_slave_1 \
--entrypoint="mesos-slave" \
-e "MESOS_MASTER=zk://${HOST_IP}:2181/mesos" \
-e "MESOS_LOG_DIR=/var/log/mesos" \
-e "MESOS_LOGGING_LEVEL=INFO" \
garland/mesosphere-docker-mesos-master
```

