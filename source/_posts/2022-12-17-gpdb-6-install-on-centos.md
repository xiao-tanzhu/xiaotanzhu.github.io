---
layout: post
title: 在CentOS上安装GreenPlum
date: 2022-12-17
tags:
- CentOS
- GreenPlum
categories: Linux
description: 在CentOS服务器上安装GreenPlum服务
---

> Refer: https://docs.vmware.com/en/VMware-Tanzu-Greenplum/6/greenplum-database/GUID-install_guide-prep_os.html

## 主机规划

|IP地址|主机名|节点类型|Server|
|-|-|-|-|
|10.0.0.100|gpdb-master|master|CentOS 8|
|10.0.0.101|gpdb-segment-01|segment|CentOS 8|
|10.0.0.102|gpdb-segment-02|segment|CentOS 8|
|10.0.0.103|gpdb-segment-03|segment|CentOS 8|

## 配置操作系统参数

### Disable SELinux

#### 查看SELinux状态：
```bash
sestatus
```

输出：
```
09:05:47 root@gpdb-master ~ → sestatus 
SELinux status:                 enabled
SELinuxfs mount:                /sys/fs/selinux
SELinux root directory:         /etc/selinux
Loaded policy name:             targeted
Current mode:                   enforcing
Mode from config file:          enforcing
Policy MLS status:              enabled
Policy deny_unknown status:     allowed
Memory protection checking:     actual (secure)
Max kernel policy version:      33
```

#### 关闭SELinux

修改配置文件：/etc/selinux/config
```ini
SELINUX=disabled
```

修改完成后重启服务器。

### 关闭iptables/firewalld

CentOS8是firewalld，检查firewalld状态：
```bash
systemctl status firewalld
```

输出：
```
09:15:27 root@gpdb-master ~ → systemctl status firewalld
● firewalld.service - firewalld - dynamic firewall daemon
   Loaded: loaded (/usr/lib/systemd/system/firewalld.service; enabled; vendor preset: enabled)
   Active: active (running) since Sun 2022-12-18 04:55:03 CST; 7h left
     Docs: man:firewalld(1)
 Main PID: 986 (firewalld)
    Tasks: 2 (limit: 49426)
   Memory: 38.2M
   CGroup: /system.slice/firewalld.service
           └─986 /usr/libexec/platform-python -s /usr/sbin/firewalld --nofork --nopid

Dec 18 04:55:01 gpdb-master systemd[1]: Starting firewalld - dynamic firewall daemon...
Dec 18 04:55:03 gpdb-master systemd[1]: Started firewalld - dynamic firewall daemon.
Dec 18 04:55:03 gpdb-master firewalld[986]: WARNING: AllowZoneDrifting is enabled. This is considere
```

关闭Firewalld：
```bash
systemctl stop firewalld.service
systemctl disable firewalld
```

### 优化系统参数（/etc/sysctl.conf）

查看当前状态：
```
sysctl -a
```

调整之后`/etc/sysctl.con`如下：
```ini
# kernel.shmall = _PHYS_PAGES / 2 # See Shared Memory Pages
# Segment的配置，通过这个命令获取：echo $(expr $(getconf _PHYS_PAGES) / 2) 
kernel.shmall = 993697
# kernel.shmmax = kernel.shmall * PAGE_SIZE 
# Segment的配置，通过这个命令获取：echo $(expr $(getconf _PHYS_PAGES) / 2 \* $(getconf PAGE_SIZE))
kernel.shmmax = 4070182912
kernel.shmmni = 4096
# kernel.shmall = 18446744073692774399
# kernel.shmmax = 18446744073692774399
# kernel.shmmni = 4096
# 固定设置为2即可
vm.overcommit_memory = 2 # See Segment Host Memory
#vm.overcommit_memory = 0
vm.overcommit_ratio = 80 # See Segment Host Memory
#vm.overcommit_ratio = 50

net.ipv4.ip_local_port_range = 10000 65535 # See Port Settings
#net.ipv4.ip_local_port_range = 32768	60999

# TODO: Kernel的几个参数还没有明确
kernel.sem = 250 2048000 200 8192
kernel.sysrq = 1
kernel.core_uses_pid = 1
kernel.msgmnb = 65536
kernel.msgmax = 65536
kernel.msgmni = 2048

# TODO: tcp的这几个优化还没有明确
net.ipv4.tcp_syncookies = 1
net.ipv4.conf.default.accept_source_route = 0
#net.ipv4.conf.default.accept_source_route = 1
net.ipv4.tcp_max_syn_backlog = 4096
#net.ipv4.tcp_max_syn_backlog = 256
net.ipv4.conf.all.arp_filter = 1
#net.ipv4.conf.all.arp_filter = 1

# UDP优化
net.ipv4.ipfrag_high_thresh = 41943040
net.ipv4.ipfrag_low_thresh = 31457280
net.ipv4.ipfrag_time = 60
#net.ipv4.ipfrag_time = 30
net.core.netdev_max_backlog = 10000
net.core.rmem_max = 2097152
net.core.wmem_max = 2097152

vm.swappiness = 10
vm.zone_reclaim_mode = 0
vm.dirty_expire_centisecs = 500
vm.dirty_writeback_centisecs = 100

# 以下四个是针对内存64GB以上的服务器
#vm.dirty_background_ratio = 0 # See System Memory
#vm.dirty_ratio = 0
#vm.dirty_background_bytes = 1610612736
#vm.dirty_bytes = 4294967296

# 针对内存64GB及以下的服务器，用以下两个替代：
vm.dirty_background_ratio = 3
#vm.dirty_background_ratio = 10
vm.dirty_ratio = 10
#vm.dirty_ratio = 30
```

## 安装软件包

### 下载

```bash
wget https://github.com/greenplum-db/gpdb/releases/download/6.22.2/open-source-greenplum-db-6.22.2-rhel8-x86_64.rpm
```

### 安装

#### 安装依赖

// TODO: To be continued..