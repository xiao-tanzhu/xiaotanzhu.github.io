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
docker pull mesoscloud/zookeeper
docker pull mesoscloud/mesos-master
docker pull fify/marathon
```
在从节点上拉取mesos-slave镜像：
```bash
docker pull mesoscloud/mesos-slave
```

### 启动主节点相关服务

**启动zookeeper**

```bash
docker run -d -e MYID=1 -e SERVERS=192.168.1.2 --name=zookeeper --net=host --restart=always mesoscloud/zookeeper
```
如果有多个节点，则`SERVERS`参数为多个IP，用“,”隔开。


**启动mesos-master**

```bash
docker run -d -e MESOS_HOSTNAME=192.168.1.2 -e MESOS_IP=192.168.1.2 -e MESOS_QUORUM=1 -e MESOS_ZK=zk://192.168.1.2:2181/mesos --name mesos-master --net host --restart always mesoscloud/mesos-master
```

**启动marathon**

```bash
docker run -d -e MARATHON_HOSTNAME=192.168.1.2 -e MARATHON_MASTER=zk://192.168.1.2:2181/mesos -e MARATHON_ZK=zk://192.168.1.2:2181/marathon --name marathon -p 8097:8080 --restart always fify/marathon
```

### 启动从节点相关服务

分别启动三个从节点：
```bash
docker run -d -e MESOS_HOSTNAME=192.168.1.2 -e MESOS_IP=192.168.1.2 -e MESOS_MASTER=zk://192.168.1.2:2181/mesos -v /sys/fs/cgroup:/sys/fs/cgroup -v /var/run/docker.sock:/var/run/docker.sock --name mesos-slave --net host --privileged --restart always mesoscloud/mesos-slave
docker run -d -e MESOS_HOSTNAME=192.168.1.3 -e MESOS_IP=192.168.1.3 -e MESOS_MASTER=zk://192.168.1.2:2181/mesos -v /sys/fs/cgroup:/sys/fs/cgroup -v /var/run/docker.sock:/var/run/docker.sock --name mesos-slave --net host --privileged --restart always mesoscloud/mesos-slave
docker run -d -e MESOS_HOSTNAME=192.168.1.4 -e MESOS_IP=192.168.1.4 -e MESOS_MASTER=zk://192.168.1.2:2181/mesos -v /sys/fs/cgroup:/sys/fs/cgroup -v /var/run/docker.sock:/var/run/docker.sock --name mesos-slave --net host --privileged --restart always mesoscloud/mesos-slave
```

