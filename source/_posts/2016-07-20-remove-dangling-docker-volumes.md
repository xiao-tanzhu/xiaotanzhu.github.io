---
layout: post
title: 删除Docker中没有被挂载的卷（dangling volumes）
date: 2016-07-20
tags:
- Docker
- Volumes
categories: 虚拟化
description: 删除Docker中没有被挂载的卷
---
最近在帮合作上制作Docker镜像。由于系统中有需要持久化的数据，数据卷几乎成了一个必选项。但是在N多次创建镜像、删除实例、新建实例的过程中，发现我的硬盘居然不够用了！查看`/var/lib/docker`的空间，居然有几十个G之多。于是想到了数据卷。

###重建实例时数据卷是否会被保留
基于同一个镜像时，多次创建实例会共享他们创建出的数据卷，这也是数据卷的最常用用法。但当镜像被修改时，即使名字和tags都相同，只要镜像ID不同，那么基于“相同名字”的这个镜像创建出的实例就不会共享之前的数据卷。

原因就在于我的系统中保留了很多过时的未被挂载的数据卷。

###删除未被挂载的数据卷
Docker并没有提供直接删除所有无用卷的功能（实际上出于安全考虑也不应该提供）。但是通过命令组合可以达到这个目的。

以下命令可以现实所有的未挂载数据卷：
```bash
docker volume ls -f dangling=true
```
组合使用以下命令可以删除所有未挂载卷：
```bash
docker volume rm $(docker volume ls -qf dangling=true)
```
