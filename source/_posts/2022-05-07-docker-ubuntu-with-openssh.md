---
layout: post
title: 构建含有OpenSSH服务器的Ubuntu基础镜像
date: 2022-05-07
tags:
- Ubuntu
- Docker
- OpenSSH
categories: Docker
description: 构建含有OpenSSH服务器的Ubuntu基础镜像
---

## 创建Dockerfile

```Dockerfile
FROM ubuntu:latest

RUN apt update && apt install vim openssh-server sudo -y

RUN useradd -rm -d /home/ubuntu -s /bin/bash -g root -G sudo -u 1000 ubuntu

RUN  echo 'ubuntu:ubuntu' | chpasswd

RUN service ssh start

EXPOSE 22

CMD ["/usr/sbin/sshd","-D"]
```

## 构建镜像

```bash
docker build -t fify/ubuntu-ssh .
```

## 启动镜像

```bash
docker run -d --name=doris-be1 --hostname=doris-be1 -p 2122:22 fify/ubuntu-with-openssh
```
