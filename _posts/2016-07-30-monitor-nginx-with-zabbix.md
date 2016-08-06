---
layout: post
title: 使用zabbix监控Nginx
tags:
- Nginx
- 监控
- Zabbix
- 转载
categories: 运维
description: 使用zabbix监控Nginx
---

#### 系统环境

- Zabbix Server：2.4.8
- ZABBIX服务端：192.168.5.254
- ZABBIX客户端：192.168.5.251
- 依赖软件包： net-tools

模板以及脚本：http://pan.baidu.com/s/1dEZMTcl

#### 配置nginx.conf
```bash
[root@master zabbix_agentd.d]# vim /etc/nginx/nginx.conf
```
添加以下内容：
```nginx
location /nginx_status {
  stub_status on;
  access_log  off;
  allow 127.0.0.1;
  allow 192.168.5.251;
  deny all;
}
```
重启服务，测试是否可以正常访问nginx_status网页

#### 添加`nginx.conf`至`/etc/zabbix/zabbix_agentd.d/`

```bash
[root@master zabbix_agentd.d]# vim /etc/zabbix/zabbix_agentd.d/nginx.conf
```
```ini
UserParameter=nginx.accepts,/etc/zabbix/zabbix_scripts/nginx_status.sh accepts
UserParameter=nginx.handled,/etc/zabbix/zabbix_scripts/nginx_status.sh handled
UserParameter=nginx.requests,/etc/zabbix/zabbix_scripts/nginx_status.sh requests
UserParameter=nginx.connections.active,/etc/zabbix/zabbix_scripts/nginx_status.sh active
UserParameter=nginx.connections.reading,/etc/zabbix/zabbix_scripts/nginx_status.sh reading
UserParameter=nginx.connections.writing,/etc/zabbix/zabbix_scripts/nginx_status.sh writing
UserParameter=nginx.connections.waiting,/etc/zabbix/zabbix_scripts/nginx_status.sh waiting
```
#### 添加`nginx_status.sh`至`/etc/zabbix/zabbix_scripts/`
内容如下
```bash
#!/bin/bash
# Author: krish@toonheart.com 
# License: GPLv2
# Set Variables 
HOST=`/sbin/ifconfig eno16777736 | grep "inet " | awk '{print $2}'`
PORT="80"
URI="nginx_status"
# Functions to return nginx stats
function active {
	/usr/bin/curl "http://$HOST:$PORT/$URI" 2> /dev/null| grep 'Active' | awk '{print $NF}'
}
function reading {
	/usr/bin/curl "http://$HOST:$PORT/$URI" 2> /dev/null| grep 'Reading' | awk '{print $2}'
}
function writing {
	/usr/bin/curl "http://$HOST:$PORT/$URI" 2> /dev/null| grep 'Writing' | awk '{print $4}'
}
function waiting {
	/usr/bin/curl "http://$HOST:$PORT/$URI" 2> /dev/null| grep 'Waiting' | awk '{print $6}'
}
function accepts {
	/usr/bin/curl "http://$HOST:$PORT/$URI" 2> /dev/null| awk NR==3 | awk '{print $1}'
}
function handled {
	/usr/bin/curl "http://$HOST:$PORT/$URI" 2> /dev/null| awk NR==3 | awk '{print $2}'
}
function requests {
	/usr/bin/curl "http://$HOST:$PORT/$URI" 2> /dev/null| awk NR==3 | awk '{print $3}'
}
# Run the requested function 
$1
```