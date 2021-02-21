---
layout: post
title: Ubuntu上使用USB安装APK到手机
tags:
- Ubuntu
- Android
- 安卓
- USB
categories: 安卓
description: Ubuntu上使用USB安装APK到手机
---

Ubuntu上可以使用`adb`命令安装APK到手机上。

## 安装adb
```bash
apt-get install android-tools-adb
```

## 查看已经连接的设备
```bash
adb devices
```
输出如下结果：
```
fify@fify-Vostro-3902:~$ adb devices
List of devices attached
ZLPFCI6L4LEUJJUG        device
```

## 安装APK包到指定设备
```bash
adb -s ZLPFCI6L4LEUJJUG install ~/Desktop/Irenshi_V3.3.1-27_release.apk
```
其中`-s`指定要安装到哪一台设备。

输出如下，当出现**Success**的时候表示安装成功，这时可以在手机上运行已经安装的程序：
```
fify@fify-Vostro-3902:~$ adb -s ZLPFCI6L4LEUJJUG install ~/Desktop/Irenshi_V3.3.1-27_release.apk
12243 KB/s (27987100 bytes in 2.232s)
        pkg: /data/local/tmp/Irenshi_V3.3.1-27_release.apk
Success
```
