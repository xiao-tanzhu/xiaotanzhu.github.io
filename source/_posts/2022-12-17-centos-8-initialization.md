---
layout: post
title: CentOS最小化安装后初始化
date: 2022-12-17
tags:
- CentOS
categories: Linux
description: CentOS最小化安装之后，软件特别少，需要做一下初始化配置。
---

## 网络

### ifconfig

```bash
yum install net-tools -y
```

### Set Static IP Addresses

配置地址：/etc/sysconfig/network-scripts/ifcfg-ens192

默认网卡设置：
```ini
TYPE=Ethernet
PROXY_METHOD=none
BROWSER_ONLY=no
BOOTPROTO=dhcp
DEFROUTE=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_FAILURE_FATAL=no
IPV6_ADDR_GEN_MODE=eui64
NAME=ens192
UUID=0c64cb51-f1bf-49b5-900d-ef6e4dc05b39
DEVICE=ens192
ONBOOT=ye
```

修改成静态地址：

```ini

```

> 其中参数的一些说明：
> 
> BOOTPROTO=<protocol> where <protocol> is one of the following:
> - none — No boot-time protocol should be used.
> - bootp — The BOOTP protocol should be used.
> - dhcp — The DHCP protocol should be used.
> 
> IPADDR=<address> where <address> is the IP address.
> DEVICE=<name> where <name> is the name of the physical device.
> DNS{1,2}=<address> where <address> is a name server address to be placed in /etc/resolv.conf
> GATEWAY=<address> where <address> is the IP address of the network router
> MACADDR=<MAC-address> Where <MAC-address> is the hardware address of the Ethernet device in the form AA:BB:CC:DD:EE:F
> NETMASK=<mask> Where <mask> is the netmask value.
> ONBOOT=<answer> Where <answer> is one of the following:
> - yes — This device should be activated at boot-time.
> - no — This device should not be activated at boot-time.
> PEERDNS=<answer> Where <answer> is one of the following:
> - yes – Modify /etc/resolv.conf if the DNS directive is set. If using DHCP, then yes is the default.
> - no – Do not modify /etc/resolv.conf.
> USERCTL=<answer> where <answer> is one of the following:
> - yes  – Non-root users are allowed to control this device.
> - no – Non-root users are not allowed to control this device.

## 主机名称

修改以下两个文件：
1. /etc/hostname
2. /etc/hosts

## Utilities

### Oh-my-bash

需要先安装Git：

```bash
yum install git -y
```

然后安装on-my-bash：

```bash
bash -c "$(curl -fsSL https://raw.githubusercontent.com/ohmybash/oh-my-bash/master/tools/install.sh)"
```

### EPEL

安装epel：
```bash
yum install epel-release -y
```

### Source List

修改镜像为上海交大镜像：
```bash
sed -e 's/mirrorlist/#mirrorlist/g' -e 's|#baseurl=http://mirror.centos.org/|baseurl=http://mirror.sjtu.edu.cn/|g' -i.bak /etc/yum.repos.d/CentOS-Stream-BaseOS.repo 
sed -e 's/mirrorlist/#mirrorlist/g' -e 's|#baseurl=http://mirror.centos.org/|baseurl=http://mirror.sjtu.edu.cn/|g' -i.bak /etc/yum.repos.d/CentOS-Stream-Extras.repo 
sed -e 's/mirrorlist/#mirrorlist/g' -e 's|#baseurl=http://mirror.centos.org/|baseurl=http://mirror.sjtu.edu.cn/|g' -i.bak /etc/yum.repos.d/CentOS-Stream-AppStream.repo 
sed -e 's/mirrorlist/#mirrorlist/g' -e 's|#baseurl=http://mirror.centos.org/|baseurl=http://mirror.sjtu.edu.cn/|g' -i.bak /etc/yum.repos.d/CentOS-Stream-Extras-common.repo 
sed -e 's/mirrorlist/#mirrorlist/g' -e 's|#baseurl=http://mirror.centos.org/|baseurl=http://mirror.sjtu.edu.cn/|g' -i.bak /etc/yum.repos.d/CentOS-Stream-Sources.repo
```
