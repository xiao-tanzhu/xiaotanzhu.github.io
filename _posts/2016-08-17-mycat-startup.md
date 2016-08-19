---
layout: post
title: 开始使用MyCAT
tags:
- MyCAT
- MySQL
- Cobar
- 分库分表
- 读写分离
- 中间件
categories: 数据库
description: MyCAT开始使用时候遇到的问题。
---
MyCAT是基于阿里巴巴开源数据库中间件Cobar开发并维护的数据库中间件，弥补了Cobar无人维护的尴尬境地。详细介绍可以参考官网：http://mycat.io/

相比Cobar来说，MyCAT的坑算是少很多了，下面是开始使用MyCAT的一些步骤。

## 准备数据库

### 创建MySQL数据库实例

用Docker启动，其他安装方式不再介绍。
```bash
docker run --name mysql-001 -e MYSQL_ROOT_PASSWORD=root -d mysql:5.6 --character-set-server=utf8mb4 --collation-server=utf8mb4_unicode_ci
```

### 修改MySQL服务器配置
```ini
[mysqld]
skip-host-cache
skip-name-resolve
user            = mysql
pid-file        = /var/run/mysqld/mysqld.pid
socket          = /var/run/mysqld/mysqld.sock
port            = 3306
basedir         = /usr
datadir         = /var/lib/mysql
tmpdir          = /tmp
lc-messages-dir = /usr/share/mysql
explicit_defaults_for_timestamp
lower-case-table-names=1
max_allowed_packet=32M
```
其中`lower-case-table-names=1`和`max_allowed_packet=32M`为非默认参数：

- `lower-case-table-names=1`忽略表格名字大小写，否则MyCAT会无法找到表格
- `max_allowed_packet=32M`该参数需要配合mysqldump，参考[使用`mysqldump`导出](#使用mysqldump导出)。

## 准备MyCAT

### 设置MyCAT虚拟schema

### 设置MyCAT读写分离

### 设置MyCAT水平切分

## 导入数据I：使用`mysqldump`+`source`命令

### 使用`mysqldump`导出

设置`mysqldump`的一些参数
```ini
[mysqldump]
quick
quote-names
max_allowed_packet      = 16M
default-character-set   = utf8mb4
```
当然这些也可以在执行`mysqldump`命令的时候指定。

- `max_allowed_packet`指定了最大允许的包大小。如果不指定则默认为24MB，导入的时候可能会报`ERROR 1153 (08S01) at line 1133809: Got a packet bigger than 'max_allowed_packet' bytes`错误，因为MySQL允许的默认大小为1MB。
- `default-character-set`保证在导出数据库的时候使用`utf8mb4`编码。关于`utf8mb4`可以参考：[创建支持emoji表情的MySQL数据库（utf8mb4）](http://www.xiaotanzhu.com/2016/08/14/create-mysql-database-with-utf8mb4.html)。

导出命令：
```bash
mysqldump -h192.168.1.3 -ulinahr -plinahr -c --skip-add-locks --skip-extended-insert --no-autocommit linahr > irenshi-data.sql
```
其中每个参数都几乎是必选项，`-h`, `-u`, `-p`三个参数不多说，下面介绍其他参数：

- `-c`，全称为`--complete-insert`，告诉mysqldump导出的时候把列名一起导出。这在MyCAT中要求是必选的，因为MyCAT在执行插入的时候只能执行带列名的插入语句
- `--skip-add-locks` 默认情况下，mysqldump会在每个表前后分别增加`LOCK TABLES`和`UNLOCK TABLES`语句，但在MyCAT中使用`LOCK TABLES `和`UNLOCK TABLES`会造成潜在的死锁风险，所以尽量避免使用
- `--skip-extended-insert`默认情况下，mysqldump会把每个表格的所有数据写到同一个SQL中。但在分库分表情况下执行导入的时候，对于Boolean类型的数据并且值为`True`的数据，会报[Data too long](https://github.com/MyCATApache/Mycat-Server/issues/1054)错误。目前还不能确认是否是MyCAT的Bug。增加该参数，将每行数据输出为一个单独的insert语句，就不会出现类似错误了。
- `--no-autocommit`参数在每个表格所有的插入语句的前后分别增加`SET autocommit = 0`和`COMMIT`语句。相比没有这个参数，插入速度能差出至少200倍，分别是10000QPS和50QPS。

**如果需要在mysqldump的时候忽略一些表格**，可以使用`--ignore-table`参数。如果需要一次性忽略一批表格，可以使用这个脚本：
```bash
#!/bin/bash

PASSWORD=linahr
HOST=192.168.1.3
USER=linahr
DATABASE=linahr
DB_FILE=linahr.sql
EXCLUDED_TABLES=(
                qrtz_blob_triggers
                qrtz_calendars
                qrtz_cron_triggers
                qrtz_fired_triggers
                qrtz_job_details
                qrtz_locks
                qrtz_paused_trigger_grps
                qrtz_scheduler_state
                qrtz_simple_triggers
                qrtz_simprop_triggers
                qrtz_triggers
                )

IGNORED_TABLES_STRING=''
for TABLE in "${EXCLUDED_TABLES@]}"
do :
	IGNORED_TABLES_STRING+=" --ignore-table=${DATABASE}.${TABLE}"
done

#echo "Dump structure"
#mysqldump --host=${HOST} --user=${USER} --password=${PASSWORD} --single-transaction --no-data ${IGNORED_TABLES_STRING} ${DATABASE} > irenshi-tables.sql

echo "Dump content"
mysqldump --host=${HOST} --user=${USER} --password=${PASSWORD} --opt -c --skip-add-locks --no-autocommit --skip-extended-insert ${DATABASE} ${IGNORED_TABLES_STRING} > irenshi-data.sql
```

### 使用`mysql`命令导入

首先需要修改`[mysqld]`的`max_allowed_packet`参数。这个参数默认为1MB，但mysqldump导出的最大允许为16MB，会造成错误，见[使用`mysqldump`导出](#使用mysqldump导出)章节

```ini
[mysqld]
max_allowed_packet=32M
```

开始导入：
```bash
mysql -ulinahr -plinahr -h192.168.1.4 -P8066 irenshi < irenshi-data.sql
```
或者也可以执行SQL命令：
```sql
source irenshi-data.sql
```

## 导入数据II：使用`mysqldump`+`LOAD DATA INFILE`命令

在某些数据库表比较大的情况下，使用以上方法导入的速度就比较难以接受了。MyCAT1.4以后还提供了类似MySQL的`LOAD DATA INFILE`命令，供导入大批量数据使用。据说这种方式比`insert`语句要快20倍。

### 同样使用`mysqldump`导出

对于小表，使用上面的导入方式还是比较方便的。我们只针对大表使用`LOAD DATA INFILE`。
```bash
mysqldump -h192.168.1.3 -uroot -proot --fields-optionally-enclosed-by='"' --fields-terminated-by=',' --tab /tmp/irenshi/ --lines-terminated-by='\n' linahr tab_sign_record_info
```
这个命令会将`irenshi`库中的`tab_sign_record_info`以文件形式导入到`/tmp/irenshi/`目录下。对于每一个导出的数据库表，将生成两个文件：tab_sign_record_info.sql和tab_sign_record_info.txt，其中tab_sign_record_info.sql存放了数据库表DDL，tab_sign_record_info.txt存放数据库表中的数据。

`--fields-optionally-enclosed-by='"'`，`--fields-terminated-by=','`和`--lines-terminated-by='\n'`分别指定了数据库文件的格式。这几个命令应和`LOAD DATA INFILE`给定的参数一致。

**执行此命令需要注意几点：**

1. `mysqldump`必须在MySQL服务器同一台主机上执行
2. 必须拥有写文件权限
3. MySQL服务器必须对给定的目录`/tmp/irenshi/`有写权限

### 使用`LOAD DATA INFILE`导入数据
```sql
LOAD DATA local INFILE '/tmp/irenshi/tab_sign_record_info.txt'
IGNORE INTO TABLE tab_sign_record_info CHARACTER SET 'utf8mb4' FIELDS TERMINATED BY ',' OPTIONALLY ENCLOSED BY '"' LINES TERMINATED BY '\n' (column1, column2, column3, ...)
```
其中：
- `local`表示从执行该命令的机器上获取文件。如果没有该参数，则从MySQL服务器上获取文件
- `IGNORE`指定了在服务器上如果已经存在了相同数据则忽略该行。还可以为`REPLACE`，表示替换已经存在的数据
- `CHARACTER SET 'utf8mb4'`指定数据库表的编码集。这里需要和数据的编码保持一致，否则可能会出现乱码甚至执行失败
- `FIELDS TERMINATED BY ',' OPTIONALLY ENCLOSED BY '"' LINES TERMINATED BY '\n'`和`mysqldump`中的参数对应
- `(column1, column2, column3, ...)`给定数据库的列。在MyCAT中必须要给定所有列，并且列的顺序要和建表时的顺序一致

**使用`local`文件加载数据时，需指定`local-infile = 1`参数**。如果不指定可能会报以下错误：
在MySQL上报以下错误：
```
ERROR 1148 (42000): The used command is not allowed with this MySQL version
```
而在MyCAT上则会报：
```
ERROR 2027 (HY000): Malformed packet
```
这个错误着实让人莫名其妙。

**使用该参数的方法有三种：**

- 直接在`mysql`命令中指定：
>mysql -h192.168.1.4 -ulinahr --local-infile=1 -plinahr -P 8066 irenshi

- 在mysql客户端的配置文件中设置：
>[client]
>local-infile = 1

- 在mysql连接之后的session中执行SQL命令:
>SET local_infile=1;

## 数据库扩容

## 监控
