---
layout: post
title: 使用Zabbix监控MySQL服务器
tags:
- Zabbix
- 监控
- MySQL
categories: 运维
description: 使用Zabbix监控MySQL服务器
---

从Zabbix 2.2开始，Zabbix官方已经支持了MySQL监控，但是MySQL监控默认是不可用的，需要经过额外的设置才可以使用。

## 创建MySQL监控帐号

首先要建立一个MySQL帐户用于Zabbix Agent登录获取MySQL状态，这个帐户不需要任何权限，因此实质上可以使用debian-sys-maint也是可以的，另外如果在被监控的机子上本身就安装有Zabbix Server，那么可以直接使用zabbix帐户（密码可以在/etc/zabbix/zabbix_server.conf中找到）。当然可以登录被监控端的MySQL新建一个帐户：

```sql
CREATE USER 'zabbix'@'%' IDENTIFIED BY 'zabbix';
FLUSH PRIVILEGES;
```

## 脚本收集MySQL数据

创建收集MySQL数据的脚本（/etc/zabbix/scripts/mysql-status.sh）：
```bash
#!/bin/bash
#use zabbix to monitor mysql status
#carl 20150316 1st

mysql=/usr/bin/mysql
var=$1
MYSQL_USER=$2
MYSQL_PASSWORD=$3
MYSQL_Host=$4
[ "${MYSQL_USER}"     = '' ] &&  MYSQL_USER=zabbix   #mysql的zabbix用户
[ "${MYSQL_PASSWORD}" = '' ] &&  MYSQL_PASSWORD=zabbix  #mysql的zabbix密码
[ "${MYSQL_Host}"     = '' ] &&  MYSQL_Host=192.168.1.3
[ "${var}" = '' ] && echo ""||${mysql} -h${MYSQL_Host} -u${MYSQL_USER} -p${MYSQL_PASSWORD} -e 'show global status'|grep -v Variable_name|grep "\b${var}\b"|awk '{print $2}'
```

## 配置Zabbix Agent

添加配置文件：/etc/zabbix/zabbix_agentd.d/userparameter_mysql.conf

```ini
UserParameter=mysql.status[*],/etc/zabbix/scripts/mysql-status.sh $1

# Flexible parameter to determine database or table size. On the frontend side, use keys like mysql.size[zabbix,history,data].
# Key syntax is mysql.size[<database>,<table>,<type>].
# Database may be a database name or "all". Default is "all".
# Table may be a table name or "all". Default is "all".
# Type may be "data", "index", "free" or "both". Both is a sum of data and index. Default is "both".
# Database is mandatory if a table is specified. Type may be specified always.
# Returns value in bytes.
# 'sum' on data_length or index_length alone needed when we are getting this information for whole database instead of a single table
UserParameter=mysql.size[*],bash -c 'echo "select sum($(case "$3" in both|"") echo "data_length+index_length";; data|index) echo "$3_length";; free) echo "data_free";; esac)) from information_schema.tables$([[ "$1" = "all" || ! "$1" ]] || echo " where table_schema=\"$1\"")$([[ "$2" = "all" || ! "$2" ]] || echo "and table_name=\"$2\"");" | HOME=/var/lib/zabbix mysql -N'

UserParameter=mysql.ping,/usr/bin/mysqladmin ping -h192.168.1.3 -uzabbix -pzabbix|grep alive|wc -l
UserParameter=mysql.version,/usr/bin/mysql -h192.168.1.3 -uzabbix -pzabbix -e "select version();"|awk 'END {print}'
```
其中192.168.1.3为MySQL服务器IP。

如果不想在脚本中使用用户名密码，可以使用`.my.cnf`配置文件，然后使用类似`HOME=/var/lib/zabbix mysql -N`命令访问MySQL，其中HOME为存放`.my.cnf`文件的路径。

## 增加配置模板

在Zabbix的Hosts中增加`Template App MySQL`这个模板，如图所示：

![](/upload/images/12.png)