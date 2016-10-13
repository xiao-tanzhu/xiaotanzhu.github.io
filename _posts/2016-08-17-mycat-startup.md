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
docker run --name mysql-001 -p 3306:3306 -e MYSQL_ROOT_PASSWORD=root -d mysql:5.6 --character-set-server=utf8mb4 --collation-server=utf8mb4_unicode_ci
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

#### 定义虚拟schema：schema.xml
```xml
<?xml version="1.0"?>
<!DOCTYPE mycat:schema SYSTEM "schema.dtd">
<mycat:schema xmlns:mycat="http://org.opencloudb/" >
        <schema name="irenshi" checkSQLschema="false" sqlMaxLimit="10000" dataNode="dn001">
                <table name="tab_sign_record_info" primaryKey="id" dataNode="dn001,dn002,dn003" rule="sharding-by-company-id" />
        </schema>
        <schema name="linahr" checkSQLschema="false" sqlMaxLimit="10000" dataNode="dn004">
                <table name="tab_web_user" primaryKey="id" dataNode="dn004,dn005" rule="sharding-by-company-id-murmur" />
        </schema>
        <dataNode name="dn001" dataHost="mysql-001" database="irenshi001" />
        <dataNode name="dn002" dataHost="mysql-001" database="irenshi002" />
        <dataNode name="dn003" dataHost="mysql-001" database="irenshi003" />
        <dataHost name="mysql-001" maxCon="1000" minCon="10" balance="0" writeType="0" dbType="mysql" dbDriver="native" switchType="1"  slaveThreshold="100">
                <heartbeat>select user();</heartbeat>
                <writeHost host="m001" url="192.168.1.4:3306" user="root" password="root">
                        <!--<readHost host="s001" url="192.168.1.4:3306" user="root" password="root" />-->
                </writeHost>
        </dataHost>
</mycat:schema>
```
**`<schema>`标签：**
- `name="irenshi"`定义的数据库名称为`irenshi`
- `checkSQLschema="false"`不对select语句中的schema名称做处理。该值置为`true`时，如果我们执行询句`select * from TESTDB.travelrecord;`则MyCat会把询句修改为`select * from travelrecord;`。即把表示schema字符去捧，避免发送到后端数据库执行时报：*（ERROR1146 (42S02): Table ‘testdb.travelrecord’ doesn’t exist）*。
- `sqlMaxLimit="10000"`在selecct语句不指定`limit`的时候，最多返回10000条数据
- `dataNode="dn001"`在不使用`<table>`指明的情况下，数据库表存放到`dn001`节点

**`<table>`标签：**
`<table>`标签不指定的数据库表均以`<schema>`的设置为准，指定的话以指定的为准。
- `name="tab_sign_record_info"`指定要设置的数据库表
- `primaryKey="id"`指定数据库表的主键。设置该值之后，如果MyCAT第一次执行主键查询时，会把请求发送到所有后端服务器，并且将主键所对应的数据库位置缓存，下次查询的时候直接根据该缓存向对应的数据库发送请求
- `dataNode="dn001,dn002,dn003"`表明该表将被存放到`dn001,dn002,dn003`三个MySQL中
- `rule="sharding-by-company-id"`给出表中的数据如何分布到上边给定的三个MySQL中

**`<dataNode>`标签：**
`<dataNode`标签定义MyCAT的数据节点。每个数据节点定位到某个MySQL主机的某个数据库schema上。

**`<dataHost>`标签：**
`<dataHost>`定义MySQL物理节点以及其连接方式。具体可以参考[《MyCAT权威指南》](http://mycat.io/document/Mycat_V1.6.0.pdf)。

### 设置MyCAT读写分离

MyCAT的读写分离通过`schema.xml`中的`<dataNode>`标签来定义。其中一个`<dataNode>`可以对应一个或者多个`<writeHost>`，而一个`<writeHost>`又可以有零个或者多个`<readHost>`。
其中`<writeHost>`之间可以互为备份，取决于`balance="0"`的设置；当一个`<writeNode>`挂掉的时候，它下边的所有`<readHost>`也不可访问。

**`balance`参数可取的值：**
- `balance="0"`, 不开启读写分离机制，所有读操作都都发送到当前可用的`writeHost`上
- `balance="1"`，全部`readHost`与`stand by writeHost`参与`select`语句的负载均衡，简单的说，当双主双从模式(M1->S1，M2->S2，并且 M1 与 M2 互为主备)，正常情冴下，M2,S1,S2 都参与`select`语句的负载均衡
- `balance="2"`，所有读操作都随机在`writeHost`、`readhost`上分发
- `balance="3"`，所有读请求随机分发到`wiriterHost`对应的`readhost`执行，`writerHost`不负担读压力，注意`balance=3`只在1.4 及其以后版本有，1.3没有

### 设置MyCAT水平切分

#### 使用字符串前缀切分

`<table>`指定了数据库表`tab_sign_record_info`的水平切分方式：
```xml
<schema name="irenshi" checkSQLschema="false" sqlMaxLimit="10000" dataNode="dn001">
    <table name="tab_sign_record_info" primaryKey="id" dataNode="dn001,dn002,dn003" rule="sharding-by-company-id" />
</schema>
```
其中的数据由`rule="sharding-by-company-id"`指定的算法切分。

在`rule.xml`中我们可以看到`sharding-by-company-id`的定义：
```xml
<tableRule name="sharding-by-company-id">
        <rule>
                <columns>companyId</columns>
                <algorithm>sharding-by-pattern</algorithm>
        </rule>
</tableRule>
<function name="sharding-by-pattern" class="org.opencloudb.route.function.PartitionByPrefixPattern">
        <property name="patternValue">64</property>
        <property name="prefixLength">5</property>
        <property name="mapFile">partition-pattern.txt</property>
</function>
```

由于CompanyID使用了UUID，为字符串类型。所以使用`PartitionByPrefixPattern`来进行计算：
- `prefixLength` 将CompanyId中的前5个字母以ASCII的方式求和
- `patternValue` 将求和之后数值MOD 64得出最终结果
- `mapFile` 将计算的最终结果按照`partition-pattern.txt`文件给定的分片规则进行分片

其中partition-pattern.txt内容如下：
```
# range start-end ,data node index
# ASCII
# 8-57=0-9 阿拉伯数字
# 64、65-90=@、A-Z
# 97-122=a-z
###### first host configuration
0-20=0
21-40=1
41-63=2
```
结合`dataNode="dn001,dn002,dn003"`设置，结果为0-20的数据将分布在dn001中，21-40的数据将分布到dn002中，41-63的数据将分布到dn003中。

#### 使用一致性哈希切分

`<table>`标签的配置如下：
```xml
<schema name="linahr" checkSQLschema="false" sqlMaxLimit="10000" dataNode="dn004">
    <table name="tab_web_user" primaryKey="id" dataNode="dn004,dn005" rule="sharding-by-company-id-murmur" />
</schema>
```
同样查看rule.xml中`sharding-by-company-id-murmur`的定义：
```xml
<tableRule name="sharding-by-company-id-murmur">
        <rule>
                <columns>companyId</columns>
                <algorithm>murmur</algorithm>
        </rule>
</tableRule>
<function name="murmur" class="org.opencloudb.route.function.PartitionByMurmurHash">
        <property name="seed">0</property>
        <property name="count">2</property>
        <property name="virtualBucketTimes">160</property>
        <!-- <property name="weightMapFile">weightMapFile</property> -->
</function>
```
其中：
- `<property name="seed">0</property>`指定了murmur算法的种子。一般不需要改，使用默认的0即可
- `<property name="count">2</property>`表示物理节点的个数，对应实际dataNode的数量。如上配置中，dataNode为`dn004,dn005`，则此处值为2
- `<property name="virtualBucketTimes">160</property>`指定一致性哈希中虚拟bucket的数量。默认为160，在这里节点数为2，那么虚拟bucket的数量为320个（假设下边介绍的weight值为默认值1）。若扩容把count的数量改为3，则虚拟bucket的数量变为480。
- `<property name="weightMapFile">weightMapFile</property>`默认值为1。每个节点对应一个weight值，假设第i个节点的weight值为`weight[i]`，则第i个节点对应的虚拟bucket数量为`weight[i]*virtualBucketTimes`。所有虚拟节点的总数为`sum(weight[i]*virtualBucketTimes)`。

当在机器中增加节点时，即增大count值时，对于每一条数据，则要么落到原有节点中、要么落到新节点中。**但是如果增大`virtualBucketTimes`或者`weight`的值，则一致性哈希的这个性质不能被保证。**所以对`virtualBucketTimes`和`weight`的修改一定要谨慎！

之前`murmur`的配置中还包含`bucketMapPath`参数，但在1.5的代码中该参数相关的代码已经被注释掉，不能使用了：https://github.com/MyCATApache/Mycat-Server/commit/c9cb201992564c315436792572e96c3beaed3b37

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

> **注意：**以下内容适合导入单个表，如果需要批量导入大量表，可以参考：http://www.xiaotanzhu.com/2016/08/24/import-data-into-mycat.html

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

*&lt;等用到的时候我再写吧&gt;*

## 监控

MyCAT官方提供了MyCAT-Eye作为监控软件。如图所示：
![](/upload/images/11.png)

启动MyCAT-Eye需要指定ZooKeeper的服务路径，修改`$MYCAT_WEB_DIR/mycat-web/WEB-INF/classes/mycat.properties`：
```diff
- zookeeper=localhost:2181
+ zookeeper=192.168.1.2:2181
```

然后进入MyCAT-Eye所在目录，执行`./start.sh`即可启动MyCAT-Eye。MyCAT-Eye默认服务路径为：
```
http://localhost:8082/mycat/
```
进入后可以对MyCAT和MySQL等进行配置。