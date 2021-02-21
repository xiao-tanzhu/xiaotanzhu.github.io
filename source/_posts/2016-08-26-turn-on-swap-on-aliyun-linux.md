---
layout: post
title: 阿里云ECS Linux开启swap（虚拟内存）
tags:
- 阿里云
- ECS
- Linux
- swap
categories: 运维
description: 阿里云ECS Linux开启swap（虚拟内存）
---

阿里云的服务器默认是不开启虚拟内存的，官方给出的理由如下：

> 由于开启swap分区会导致硬盘IO性能下降，因此阿里云服务器初始状态未配置swap。

这确实是个理由，开启swap对阿里云并没有什么太多的好处：

1. 开启swap分区会造成阿里云宿主机的磁盘负载增大，其一会影响磁盘IO性能，其次还会降低磁盘使用寿命；
2. 在阿里云推出SSD之后，SSD的性能和内存性能的差距相对机械硬盘来说减小很多，SSD设置可以当内存使用。但是SSD和内存的价格差别不是一点半点。

但是不开启swap，会造成很严重的后果：

1. 内存使用波动的时候会导致系统直接杀死某些进程，造成严重的系统不稳定因素，触发只是因为系统偶尔的一次波动而已。

## 开启swap的方法

### 创建用于交换分区的文件
```bash
dd if=/dev/zero of=/mnt/swap bs=1M count=4096
```
注：`block_size`、`number_of_block`大小可以自定义，比如bs=1M count=4096代表设置4G大小swap分区

### 设置交换分区文件

```bash
mkswap /mnt/swap
```

### 立即启用交换分区文件

```bash
swapon /mnt/swap
```
如果在/etc/rc.local中有swapoff -a 需要修改为swapon -a

### 设置开机时自启用swap分区

需要修改文件/etc/fstab中的swap行。添加以下内容：

```
/mnt/swap swap swap defaults 0 0
```
注：/mnt/swap 路径可以修改，可以根据创建的swap文件具体路径来配置。

设置后可以执行free -m命令查看效果：
```
root@www:~# free -m
             total       used       free     shared    buffers     cached
Mem:          7983       7784        199          2         70       1158
-/+ buffers/cache:       6555       1428
Swap:         4095       1052       3043
```
