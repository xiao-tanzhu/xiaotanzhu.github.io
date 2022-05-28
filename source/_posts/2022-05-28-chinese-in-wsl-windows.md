---
layout: post
title: （英文版）Windows 10 WSL 2中文乱码问题解决
date: 2022-05-28
tags:
- Ubuntu
- Windows
- WSL
categories: Windows
description: 英文版Windows 10中，安装WSL和Ubuntu之后，终端中显示的中文都变成了豆腐。这篇文档介绍如何解决这个问题。
---

## 环境

- Windows 10
- WSL 2
- Ubuntu: 20.04

## 现象

如图：终端中所有中文都变成了豆腐。

![](/images/0050.png)

## 解决办法

进入Windows设置（`Settings`）

![](/images/0055.png)

选择时间和语言（`Time & Language`），并选择语言（`Language`）

![](/images/0051.png)

点击右上角：`Administratative language settings`

![](/images/0052.png)

选择`Administrative`

![](/images/0053.png)

将`Current system locales`调整为`Chinese(Simplified, China)`

**不要勾选：`Beta: Use Unicode UTF-8 for worldwide language support`**

![](/images/0054.png)

重启系统即可。
