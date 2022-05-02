---
layout: post
title: 上海电信桥接模式下使用IPTV
date: 2021-07-02
tags: ["Merlin", "AC88U", "IPTV"]
categories: Merlin
description: 上海电信桥接模式下使用IPTV，通过Merlin版AC88U路由器
---

## 原理

上海电信机顶盒本质是一个安卓盒子，通过获取DHCP地址的方式连接网络，和一台普通电脑或电视盒子基本一样。不同点在于电信机顶盒在连接网络时有一个认证过程。

本文即通过模拟上海电信光猫对机顶盒的认证过程，来让桥接的路由器可以顺利看IPTV。

## 认证参数

在电信机顶盒获取IP地址的时候，电信光猫会附带一系列`DHCP-Options`，连同IP地址一起发送至机顶盒。

> 我家是ZTE机顶盒，获取到的`DHCP-Options`如下，其他品牌机顶盒可通过Wireshark或其他软件，监听DHCP包获取。

```
dhcp-option-force=125,00:00:00:00:1d:02:06:48:47:57:2d:43:54:03:07:48:47:32:32:30:47:53:0a:02:20:00:0b:02:00:55:0d:02:00:2e
```

## 网络拓扑

以下是我家的网络拓扑，路由器到光猫采用桥接的方式，路由器中拨号登录PPPoE。
![](/images/0049.png)

## 配置过程

### 配置光猫桥接（如果已配置请跳过）

首先将INTERNET设置为桥接模式，这里千万不要绑定任何端口，否则后面OTHER设置时候会无法绑定VLAN

![](/images/0014.jpeg)

设置IPTV的连接，这里绑定端口的是传统IPTV无需B面认证的IPTV，没有就不用勾，主要是下面的绑定数据，在你路由器所用端口打钩，vlan_1填写85，如图

![](/images/0015.jpeg)

进入应用-日常应用-IPTV 设置组播vlan为51，注意是OTHER的组播

![](/images/0016.jpeg)

### 配置DHCP服务器

我的AC88U路由器使用了梅林固件，并安装了[科学上网插件](https://github.com/hq450/fancyss)，DHCP的参数由这个插件控制，故需要修改插件相关脚本。

#### SSH登录路由器，修改dnsmasq.conf文件

路由器中dnsmasq.conf文件位置为：/tmp/etc/dnsmasq.conf。但此文件是在路由器启动的时候动态生成的，故需要修改插件中相关脚本文件：

修改文件：/jffs/scripts/dnsmasq.postconf，增加以下内容到文件最后：

```shell
# 通过 pc_append把dhcp相关的设置append到 /tmp/etc/dnsmasq.conf 文件里
# 这样IPTV就能获取到VLAN的IP了
pc_append "dhcp-option-force=125,00:00:00:00:1d:02:06:48:47:57:2d:43:54:03:07:48:47:32:32:30:47:53:0a:02:20:00:0b:02:00:55:0d:02:00:2e" /tmp/etc/dnsmasq.conf
```

#### 重启路由器

重启路由器，在/tmp/etc/dnsmasq.conf文件中可以看到以上添加的`dhcp-options`。

> **特别注意：**本文中的dnsmasq是通过[科学上网插件](https://github.com/hq450/fancyss)控制的，故关闭[科学上网插件](https://github.com/hq450/fancyss)后，该设置将失效。

## 验证

使用电脑连接路由器，并用wireshark抓DHCP包，能看到这个dhcp-options即可。

将IPTV机顶盒用网线接入到路由器上，打开即可享受IPTV了。
