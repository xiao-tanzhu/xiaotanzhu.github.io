---
layout: post
title: 设置VIM的编码格式
date: 2016-10-10
tags:
- Linux
- 编码
- vim
categories: Linux
description: 设置vim的编码格式
---

有的时候使用vim打开中文文件的时候会出现乱码，原因是vim打开文件时使用的编码格式和文件的编码格式不一致。

## 设置正确的编码格式以正确的显示中文

如果该文档使用utf8编码，则可以通过以下vim命令将编码格式设置为utf8：
```vim
set encoding=utf-8
```
这样本来以乱码显示的中文就可以恢复本来面貌。

## 设置VIM保存文件时的编码格式

为保证VIM编辑器保存的文件以utf8编码格式存储，可以先执行以下命令，然后再进行保存：
```vim
set fileencodings=utf-8
w
```

## 设置VIM默认编码格式

将以上两个命令加入到`~/.vimrc`文件中，则可以是VIM默认以utf8编码打开文件，并且以utf8编码存储文件：
```diff
+set encoding=utf-8
+set fileencodings=utf-8
```
