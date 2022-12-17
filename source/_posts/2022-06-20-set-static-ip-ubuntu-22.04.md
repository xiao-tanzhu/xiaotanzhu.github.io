---
layout: post
title: 为Ubuntu 22.04设置静态IP（NetPlan）
date: 2022-06-20
tags:
- Ubuntu
- NetPlan
categories: Linux
description: 使用NetPlan工具，为Ubuntu 22.04设置静态IP
---

在 Ubuntu 服务器 22.04 中，网络由 netplan 程序控制，因此我们将使用 netplan 在 Ubuntu 服务器上配置静态 IP 地址。

## 修改netplan配置文件

修改：`/etc/netplan/​00-installer-config.yaml`配置文件：

默认配置如下：

```yaml
# This is the network config written by 'subiquity'
network:
  ethernets:
    ens160:
      dhcp4: true
      dhcp-identifier: mac
  version: 2
```

按照需求修改如下：

```yaml
# This is the network config written by 'subiquity'
network:
  ethernets:
    ens160:
      addresses:
        - 192.168.31.123/24
      nameservers:
        addresses: [114.114.114.114, 8.8.8.8]
      routes:
        - to: default
          via: 192.168.31.1
      dhcp-identifier: mac
    ens192:
      addresses:
        - 10.0.0.100/24
  version: 2
```

## 应用新的配置

```bash
sudo netplan apply
```

## 确认生效

```bash
ifconfig
```

输出
```
05:11:35 ubuntu@gpdb-master ~ → ifconfig
docker0: flags=4099<UP,BROADCAST,MULTICAST>  mtu 1500
        inet 172.17.0.1  netmask 255.255.0.0  broadcast 172.17.255.255
        ether 02:42:d7:ee:97:02  txqueuelen 0  (Ethernet)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

ens160: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.31.123  netmask 255.255.255.0  broadcast 192.168.31.255
        inet6 fe80::250:56ff:feb1:b68  prefixlen 64  scopeid 0x20<link>
        ether 00:50:56:b1:0b:68  txqueuelen 1000  (Ethernet)
        RX packets 2707  bytes 767679 (767.6 KB)
        RX errors 0  dropped 4  overruns 0  frame 0
        TX packets 940  bytes 136029 (136.0 KB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

ens192: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 10.0.0.100  netmask 255.255.255.0  broadcast 10.0.0.255
        inet6 fe80::20c:29ff:fe93:97bb  prefixlen 64  scopeid 0x20<link>
        ether 00:0c:29:93:97:bb  txqueuelen 1000  (Ethernet)
        RX packets 2  bytes 120 (120.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 8  bytes 676 (676.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 84  bytes 6368 (6.3 KB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 84  bytes 6368 (6.3 KB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
``` 

```bash
route -n
```

输出
```
05:12:50 ubuntu@gpdb-master ~ → route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
0.0.0.0         192.168.31.1    0.0.0.0         UG    0      0        0 ens160
10.0.0.0        0.0.0.0         255.255.255.0   U     0      0        0 ens192
172.17.0.0      0.0.0.0         255.255.0.0     U     0      0        0 docker0
192.168.31.0    0.0.0.0         255.255.255.0   U     0      0        0 ens160
```
