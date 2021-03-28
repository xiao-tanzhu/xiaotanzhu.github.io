---
layout: post
title: "Linux find命令高阶用法"
date: 2021-03-28
tags: ["Linux", "Shell"]
categories: [Linux]
description: "Find命令是程序员每天都会使用的命令，是一个无处不在是命令，是linux中最有用的命令之一。你可以使用它在任意一个目录（及子目录）中搜索文件，你也可以定义一些特定的条件，如按文件名、文件类型、用户甚至是时间节点去查找文件。本文大家分享几个 find 命令的简单却又高级的用法"
---

> linux 下一切皆文件。

Find命令是程序员每天都会使用的命令，是一个无处不在是命令，是linux中最有用的命令之一。你可以使用它在任意一个目录（及子目录）中搜索文件，你也可以定义一些特定的条件，如按文件名、文件类型、用户甚至是时间节点去查找文件。

毋庸置疑的是在强大的功能背后，它的使用对比其他的命令会复杂的多，比较难。下面就跟大家分享几个 find 命令的简单却又高级的用法。

## 根据访问/修改/更改时间查找文件

如果服务器被入侵，你可以利用find 查询到近期被访问、修改、更改的文件

> 备注：min=分钟 time=天 修改注重于对内容的修改，更改注重于对权限的更改

查找1个小时内被访问过的文件

```shell
find . -amin -60
```

查找1天内被访问过的文件

```shell
find / -atime -1
```

查找在1个小时内被修改的文件

```shell
find . -mmin -60
```

查找在1天内被修改的文件

```shell
find / -mtime -1
```

查找1小时内状态被改变的文件

```shell
find . -cmin -60
```

查找1天内状态被改变的文件

```shell
find / -ctime -1
```

## 查找比某文件新或某文件旧的文件

环境上日志文件数不胜数，想删除某个时间之前的文件，该怎么处理？

> 备注：newer（修改时间）、anewer（访问时间）、ctime（修改时间，包括权限属性的修改）
列出比1.log更旧的文件

```shell
find ./ ! -newer 1.log |xargs ls -al
```

列出比1.log更新的文件

```shell
find ./ -newer 1.log |xargs ls -al
```

## 多条件组合查找

有时我们要查找的文件并不止一个类目这个时候我们可以使用多条件组合的方式去查找，常用的条件组合参数有`-a(and)`,`-o(or)`,`!(not)`。

查找普通文档和符号链接文档：

```shell
find ./ -type f -o -type l
```

查找名称为skill的符号链接文档

```shell
find ./ -name "*skill" -a -type l
```

查找log文档以外的其他文档：

```shell
find ./ ! -name "*.log"
```

## 对所查找到的文件进行操作

文件我们已经查找到了，如何对它们做些什么呢？

对于以查找到的文件，进行操作万能公式：

```shell
find . -name "*something*" -exec action {} somearguments \\;
```

> 命令解释：
> - find . -name "something" 找出所有名字包含something的文件;
> - -exec 执行后面的命令， action 任意命令名; {}是find的结果集合
> - somearguments ， 命令需要的参数，就是例子中的-r; \\; 结束命令

**举例:**

利用Find命令对文件进行备份

```shell
find . -name "*something*" –exec cp {} /backup/{}.backup /;
```

利用Find命令对文件进行删除

```shell
find . -name "*something*" –exec rm –I {} /;
```

下面列一些比较常用的：

- rm 命令，用于删除find查找出来的文件
- mv 命令，用于重命名查找出的文件
- ls -l 命令，显示查找出的文件的详细信息
- md5sum， 对查找出的文件进行md5sum运算，可以获得一个字符串，用于检测文件内容的合法性
- wc 命令，用于统计计算文件的单词数量，文件大小等

执行任何Unix的shell命令
执行你自己写的shell脚本，参数就是每个查找出来的文件名
