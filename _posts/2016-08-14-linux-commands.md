---
layout: post
title: Linux常用命令
tags:
- Shell
- Script
categories: Linux
description: Linux常用命令，包含Ubuntu等
---

## 系统相关

### 显示当前内核版本
命令：
```
uname -a
```
输出：
```
fify@fify-PC:~$ uname -a
Linux fify-PC 4.4.0-31-generic #50-Ubuntu SMP Wed Jul 13 00:07:12 UTC 2016 x86_64 x86_64 x86_64 GNU/Linux
```

### 显示当前Ubuntu的系统版本

命令：

```bash
lsb_release -a
```

输出：
```
fify@fify-PC:~$ lsb_release -a
No LSB modules are available.
Distributor ID: Ubuntu
Description:    Ubuntu 16.04.1 LTS
Release:        16.04
Codename:       xenial
```

## 用户和组

### 显示当前登录的用户名

命令：
```bash
whoami
```
输出：
```
fify@fify-PC:~$ whoami
fify
```
这个命令在组合其他命令一起使用的时候会比较好用，比如：
```bash
sudo usermod -aG docker $(whoami)
```

### 查看用户所属的组

#### 查看当前用户所属的组
命令：
```bash
groups
```
输出：
```
fify@fify-PC:~$ groups
fify adm cdrom sudo dip plugdev lpadmin sambashare
```

#### 查看其他用户所属的组
命令：
```bash
groups fify
```
输出：
```
fify@fify-PC:~$ groups fify
fify : fify adm cdrom sudo dip plugdev lpadmin sambashare docker
```