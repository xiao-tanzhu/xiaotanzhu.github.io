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
#### 在`sed`正则表达式匹配中使用Lazy策略：
`sed`命令的正则表达式并不支持懒匹配，但是我们可以通过绕过的方法来做。比如我要查找以下内容中"和"之间的单词：
```
  "departmentId" bigint(20) NOT NULL,
  "departmentName" varchar(128) COLLATE utf8mb4_unicode_ci DEFAULT NULL,
  "monthly" varchar(10) COLLATE utf8mb4_unicode_ci NOT NULL,
  "positionName" varchar(100) COLLATE utf8mb4_unicode_ci DEFAULT NULL,
  "staffId" varchar(40) COLLATE utf8mb4_unicode_ci NOT NULL,
```
那么我们可以通过`"[^\"]*"查找""之间的内容。如下：
```bash
sed -e 's/"\([^"]*\)"[^,]*,/\1,/g'
```
可以获取到：
```
  "departmentId",
  "departmentName",
  "monthly",
  "positionName",
  "staffId",
```

### 文件工具

#### 查看文件类型

```bash
file xxxx.jpg
```

`file`命令不跟据文件后缀名判断文件类型，而是根据文件最头部的几位Magic Number进行判断。对于乱改文件后缀的情况非常适合。

如：
```
fify@fify-Vostro-3902:~/Desktop$ file 1945300044.png 
1945300044.png: JPEG image data
```