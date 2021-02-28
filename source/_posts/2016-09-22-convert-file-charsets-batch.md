---
layout: post
title: 批量转换文件编码
date: 2016-09-22
tags:
- Linux
- 编码
- enca
categories: Linux
description: 批量转换文件编码
---

Linux上可以使用`enca`查看和转换文件编码。

## 查看文件编码

### 语法
```
enca [-L LANGUAGE] [FILE]...
```

### 举例

以下命令可以查看一个中文文档采用了什么编码格式。
```bash
enca -L zh_CN file.txt
```
输出如下：
```
fify@fify-Vostro-3902:~$ enca -L zh_CN password.txt
Universal transformation format 8 bits; UTF-8
```

### 特别注意

这个命令并不能100%正确的检测到文件编码，当文件中的汉字较少的时候就可能无法判断文件编码格式。（由此可见，`enca`命令也是根据文件中出现的特殊文字的编码范围“猜测”文件编码格式的。）

无法判断编码时输出如下：
```
./src/main/res/layout/activity_base_time_range_layout.xml: Unrecognized encoding
```

## 批量转换文件编码

### 语法
```
enca [-L LANGUAGE] -x <charset> [FILE]
```

### 举例

单个转换一个文件的编码时：
```
enca -L zh_CN -x UTF-8 file.txt
```
批量转换某个目录下所有`.java`文件的编码：
```
find . -name *.java | xargs enca -L zh_CN -x UTF-8
```
`enca`还有一个好处就是如果文件本来就是你要转换的那种编码，它不会报错，还是会print出结果来。

### 特别注意

虽然`enca`会跳过已经是目标编码的文件，但是多次转换同一个文件还是有出错的风险，转换完成之后记得检查。