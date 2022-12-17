---
layout: post
title: Ubuntu逻辑分区（LVM）扩容
date: 2022-12-17
tags:
- Ubuntu
- LVM
categories: Linux
description: 默认情况下，Ubuntu的LVM只用了一部分硬盘（不知道为啥这个设定），这里记录如何让LVM启用所有硬盘空间。
---

刚装完的ubuntu系统，逻辑分区容量远小于分配的磁盘容量，ubuntu逻辑分区只有50G，实际硬盘100G。可以通过下面的操作使得ubuntu逻辑分区占满整个磁盘。

## 查看LVM分区信息

```bash
lvdisplay
```

显示信息如下：
```
09:48:44 ubuntu@ansible ~ → sudo lvdisplay
  --- Logical volume ---
  LV Path                /dev/ubuntu-vg/ubuntu-lv
  LV Name                ubuntu-lv
  VG Name                ubuntu-vg
  LV UUID                cBNjoF-Suwj-90ie-7faP-lzCg-VfG3-7C0gAU
  LV Write Access        read/write
  LV Creation host, time ubuntu-server, 2022-07-17 08:04:22 +0800
  LV Status              available
  # open                 1
  LV Size                <49.00 GiB
  Current LE             12543
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           253:0
```

可以看到，`LV Size`为`<49.00 GiB`

## 查看硬盘信息

```bash
fdisk -l
```

输出如下：

```
09:48:49 ubuntu@ansible ~ → sudo fdisk -l
Disk /dev/sda: 100 GiB, 107374182400 bytes, 209715200 sectors
Disk model: Virtual disk    
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: 8AFB2AF0-3521-4611-8D54-01E6DF0EF00D

Device       Start       End   Sectors Size Type
/dev/sda1     2048      4095      2048   1M BIOS boot
/dev/sda2     4096   4198399   4194304   2G Linux filesystem
/dev/sda3  4198400 209713151 205514752  98G Linux filesystem


Disk /dev/mapper/ubuntu--vg-ubuntu--lv: 49 GiB, 52609155072 bytes, 102752256 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
```

可以看到硬盘是100GB，实际LV只用了49GB。

## 执行扩容

执行以下命令，注意`/dev/ubuntu-vg/ubuntu-lv`是第一步中获取到的`LV Path`
```bash
lvextend -l +100%FREE /dev/ubuntu-vg/ubuntu-lv
```
输出如下：
```
09:54:51 ubuntu@ansible ~ → sudo lvextend -l +100%FREE /dev/ubuntu-vg/ubuntu-lv
  Size of logical volume ubuntu-vg/ubuntu-lv changed from <49.00 GiB (12543 extents) to <98.00 GiB (25087 extents).
  Logical volume ubuntu-vg/ubuntu-lv successfully resized.
```
可以看到，`LV Size`已经由`<49.00 GiB`变成了`<98.00 GiB`

## 刷新逻辑卷LV，重新调整大小

```bash
resize2fs /dev/ubuntu-vg/ubuntu-lv
```
输出如下：
```
09:56:55 ubuntu@ansible ~ → sudo resize2fs /dev/ubuntu-vg/ubuntu-lv
resize2fs 1.46.5 (30-Dec-2021)
Filesystem at /dev/ubuntu-vg/ubuntu-lv is mounted on /; on-line resizing required
old_desc_blocks = 7, new_desc_blocks = 13
The filesystem on /dev/ubuntu-vg/ubuntu-lv is now 25689088 (4k) blocks long.
```

## 确认结果

```bash
df -lah
```

输出如下：
```
09:57:02 ubuntu@ansible ~ → sudo df -lah
Filesystem                         Size  Used Avail Use% Mounted on
sysfs                                 0     0     0    - /sys
proc                                  0     0     0    - /proc
udev                               935M     0  935M   0% /dev
devpts                                0     0     0    - /dev/pts
tmpfs                              198M  1.3M  197M   1% /run
/dev/mapper/ubuntu--vg-ubuntu--lv   97G  9.2G   83G  11% /
securityfs                            0     0     0    - /sys/kernel/security
tmpfs                              989M   84K  989M   1% /dev/shm
tmpfs                              5.0M     0  5.0M   0% /run/lock
```

可以看到`/dev/mapper/ubuntu--vg-ubuntu--lv`已经变成了97GB。
