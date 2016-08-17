---
layout: post
title: Linux常用命令
tags:
- Shell
- Script
categories: Linux
description: Linux常用命令，包含Ubuntu等
---
**注意：**常用命令中，会有一些命令仅限于特定的操作系统使用，如Ubuntu或CentOS等特定的Linux发行版本。

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

可以查看当前用户的组或者指定用户所属的组。

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

#### 以其他用户的身份执行命令
以下命令以jenkins用户的身份执行`jstack 36730`命令。
```bash
sudo -u jenkins -H jstack 36730
```

## 实用工具

### 文字处理

#### 替换某行中间的文字
```bash
sed -e 's/\(.*\)wrapper.daemonize=FALSE\(.*\)/\1wrapper.daemonize=TRUE\2/g' -i mycat
```
替换mycat文件中包含`wrapper.daemonize=FALSE`的行，并把`wrapper.daemonize=FALSE`替换为`wrapper.daemonize=TRUE`。

#### 查看文件的第m-n行
```bash
sed -n '5,10p' filename
```
这样你就可以只查看文件的第5行到第10行。

#### 删除文件的某一行

删除文件的第三行：
```bash
sed -i '3d' 1.txt
```
删除文件的第三至第五行：
```bash
sed -i '3,5d' 1.txt
```
删除符合特定正则表达式的行：
```bash
sed -i '/^Love/d' 1.txt
```
