---
layout: post
title: 将现有数据库的数据导入MyCAT
tags:
- MySQL
- MyCAT
- mysqldump
categories: 数据库
description: 将现有数据库的数据导入MyCAT
---

## 版本及其他信息

- MySQL： 5.6
- 数据库名：linahr
- MySQL Node 1: 192.168.1.3:3307
- MySQL Node 2: 192.168.1.3:3308
- MyCat Data: 192.168.1.3:8066
- MyCat Console: 192.168.1.3:9066

## 导出数据

### 修改mysqldump配置

在/etc/mysql/my.cnf中增加以下内容：
```ini
[mysqldump]
quick
quote-names
max_allowed_packet      = 16M
default-character-set   = utf8mb4
```

### 导出数据
创建/tmp/irenshi目录并修改其访问权限为777：
```bash
mkdir /tmp/irenshi
chmod 777 /tmp/irenshi
```
执行mysqldump命令导出数据：
```bash
mysqldump -uroot -proot --fields-optionally-enclosed-by='"' --fields-terminated-by=',' --tab /tmp/irenshi/ --lines-terminated-by='\n' linahr
```
数据将导出到/tmp/irenshi目录下。每个表格有两个文件，SQL(*.sql)和数据文件(*.txt)。

## 导入数据

### 合并建表语句
```bash
cat *.sql > irenshi-tables.sql
```

### 在MyCAT中创建表格
```bash
mysql -uirenshi -pirenshi -h192.168.1.3 -P8066 irenshi < irenshi-tables.sql
```

### 生成LOAD DATA语句

为保险期间，删除刚才用到的建表语句：irenshi-tables.sql
```bash
rm irenshi-tables.sql
```
在/tmp/irenshi目录下增加以下两个Shell脚本：
**read-columns.sh**，该文件会把每个SQL建表语句中的列名信息抽取出来：
```bash
#!/bin/bash

sed -e 's#^\s*`\([^`]*\)`.*,#\1, #g' $1 | sed ':label;N;s/\n//;b label' | sed -e "s/.*CREATE TABLE[^(]*(\(.*\)PRIMARY.*/\1/g" | sed -e "s/\(.*\),/\1/g"
```
**generate-sql.sh**，该文件会根据每个建表语句生成对应的LOAD DATA语句：
```bash
#!/bin/bash

OUTPUT_FILE=load-data.sql

for FILE in $(ls tab*.sql) ; do
        echo $FILE
        TABLE_NAME=`echo $FILE | cut -d "." -f1`
        COLUMNS=$(./read-columns.sh $FILE)
        #LOAD DATA local INFILE '/tmp/irenshi/tab_sign_record_info.txt'
        #IGNORE INTO TABLE tab_sign_record_info CHARACTER SET 'utf8mb4' 
        #FIELDS TERMINATED BY ',' OPTIONALLY ENCLOSED BY '"' LINES TERMINATED BY '\n' (column1, column2, column3, ...)
        SQL="LOAD DATA local INFILE '/tmp/irenshi/${TABLE_NAME}.txt'"
        SQL="${SQL} IGNORE INTO TABLE ${TABLE_NAME} CHARACTER SET 'utf8'"
        SQL="${SQL} FIELDS TERMINATED BY ',' OPTIONALLY ENCLOSED BY '\"' LINES TERMINATED BY '\n'"
        SQL="${SQL} (${COLUMNS});"
        echo $SQL >> load-data.sql
done
```
在/tmp/irenshi目录下执行generate-sql.sh，生成LOAD DATA文件：load-data.sql

### 导入数据

使用以下命令登录mysql，需要特别注意添加`--local-infile=1`参数：
```bash
mysql -uirenshi -pirenshi -h192.168.1.3 --local-infile=1 -P8066 irenshi
```
执行导入命令：
```sql
source load-data.sql
```
