---
layout: post
title: GreenPlum单节点版安装
date: 2022-06-11
tags:
- GreenPlum
categories: BigData
description: 安装单节点版GreenPlum
---

## GPDB简介

> Pivotal Greenplum Database is a MPP (massively parallel processing) database built on open source [PostgreSQL](https://en.wikipedia.org/wiki/PostgreSQL?spm=a2c6h.12873639.article-detail.4.1bcc51349E2X2H).The system consists of a master node, standby master node, and segment nodes.

> All of the data resides on the segment nodes and the catalog information is stored in the master nodes. Segment nodes run one or more segments, which are modified PostgreSQL database instances and are assigned a content identifier.

> For each table the data is divided among the segment nodes based on the distribution column keys specified by the user in the DDL statement.

> For each segment content identifier there is both a primary segment and mirror segment which are not running on the same physical host.

> When a SQL query enters the master node, it is parsed, optimized and dispatched to all of the segments to execute the query plan and either return the requested data or insert the result of the query into a database table.

## 安装GreenPlum

### Ubuntu 16.04 & 18.04

#### 安装GreenPlum

如果是Ubuntu 16.04或Ubuntu 18.04，执行以下命令安装：

```bash
sudo apt update
sudo apt install software-properties-common
sudo add-apt-repository ppa:greenplum/db
sudo apt update
sudo apt install greenplum-db-6
```

安装之后可以看到目录：
```bash
ubuntu@8837967a36f4:~$ ls /opt
greenplum-db-6.20.5
```

#### 将安装的可执行文件加载至环境变量

```bash
ubuntu@8837967a36f4:~$ source /opt/greenplum-db-6.20.5/greenplum_path.sh
ubuntu@8837967a36f4:~$ which gpssh
/opt/greenplum-db-6.20.5/bin/gpssh
```

#### 修改配置文件

首先复制一个样例配置文件到目录中：
```
ubuntu@8837967a36f4:~$ cp $GPHOME/docs/cli_help/gpconfigs/
gpinitsystem_singlenode .
```

创建hostlist文件，只包含本机名称：

```bash
echo `hostname` > hostlist_singlenode
```

设置数据目录：

创建目录：

```
mkdir -p /home/ubuntu/gpdb/primary
```

修改配置文件：

```
declare -a DATA_DIRECTORY=(/home/ubuntu/gpdb/primary /home/ubuntu/gpdb/primary)
```

> 括号中重复的次数，代表了分片的次数。

修改HOSTNAME参数为本地hostname

```
MASTER_HOSTNAME=8837967a36f4
```

修改Master数据地址：

创建目录：

```
mkdir -p /home/ubuntu/gpdb/master
```

修改配置文件：
```
MASTER_DIRECTORY=/home/ubuntu/gpdb/master
```

#### 初始化数据库

执行下以下命令，交换下ssh指纹（其实就是在known_hosts里边增加了localhost的记录）：

```
ssh localhost
```

登录之后再退出就可以了。

设置非密码登录：

```
ssh-keygen -t rsa
cp .ssh/id_rsa.pub .ssh/authorized_keys
```

执行以下命令，确认Key没有问题：

```bash
gpssh-exkeys -h localhost
```

输出如下：

```
ubuntu@8837967a36f4:~$ gpssh-exkeys -h localhost
[STEP 1 of 5] create local ID and authorize on local host
  ... /home/ubuntu/.ssh/id_rsa file exists ... key generation skipped

[STEP 2 of 5] keyscan all hosts and update known_hosts file

[STEP 3 of 5] retrieving credentials from remote hosts
  ... send to localhost

[STEP 4 of 5] determine common authentication file content

[STEP 5 of 5] copy authentication files to all remote hosts
  ... finished key exchange with localhost

[INFO] completed successfully
```

安装en_US.UTF-8编码，执行以下命令，按提示选择即可：

```
sudo dpkg-reconfigure locales
```

执行以下命令执行数据库初始化：

```
gpinitsystem -c gpinitsystem_singlenode
```

输出如下：

```
ubuntu@8837967a36f4:~/gpdb$ gpinitsystem -c gpinitsystem_singlenode
/usr/bin/locale: Cannot set LC_CTYPE to default locale: No such file or directory
/usr/bin/locale: Cannot set LC_MESSAGES to default locale: No such file or directory
/usr/bin/locale: Cannot set LC_COLLATE to default locale: No such file or directory
20220611:12:20:16:005363 gpinitsystem:8837967a36f4:ubuntu-[INFO]:-Checking configuration parameters, please wait...
20220611:12:20:16:005363 gpinitsystem:8837967a36f4:ubuntu-[INFO]:-Reading Greenplum configuration file gpinitsystem_singlenode
20220611:12:20:16:005363 gpinitsystem:8837967a36f4:ubuntu-[INFO]:-Locale has not been set in gpinitsystem_singlenode, will set to default value
/usr/bin/locale: Cannot set LC_CTYPE to default locale: No such file or directory
/usr/bin/locale: Cannot set LC_MESSAGES to default locale: No such file or directory
/usr/bin/locale: Cannot set LC_COLLATE to default locale: No such file or directory
20220611:12:20:17:005363 gpinitsystem:8837967a36f4:ubuntu-[INFO]:-Locale set to en_US.utf8
/usr/bin/locale: Cannot set LC_CTYPE to default locale: No such file or directory
/usr/bin/locale: Cannot set LC_MESSAGES to default locale: No such file or directory
/usr/bin/locale: Cannot set LC_ALL to default locale: No such file or directory
20220611:12:20:17:005363 gpinitsystem:8837967a36f4:ubuntu-[INFO]:-No DATABASE_NAME set, will exit following template1 updates
20220611:12:20:17:005363 gpinitsystem:8837967a36f4:ubuntu-[INFO]:-MASTER_MAX_CONNECT not set, will set to default value 250
20220611:12:20:17:005363 gpinitsystem:8837967a36f4:ubuntu-[INFO]:-Detected a single host GPDB array build, reducing value of BATCH_DEFAULT from 60 to 4
20220611:12:20:17:005363 gpinitsystem:8837967a36f4:ubuntu-[WARN]:-Master open file limit is 1024 should be >= 65535
20220611:12:20:17:005363 gpinitsystem:8837967a36f4:ubuntu-[INFO]:-Checking configuration parameters, Completed
20220611:12:20:17:005363 gpinitsystem:8837967a36f4:ubuntu-[INFO]:-Commencing multi-home checks, please wait...
.
20220611:12:20:17:005363 gpinitsystem:8837967a36f4:ubuntu-[INFO]:-Configuring build for standard array
20220611:12:20:17:005363 gpinitsystem:8837967a36f4:ubuntu-[INFO]:-Commencing multi-home checks, Completed
20220611:12:20:17:005363 gpinitsystem:8837967a36f4:ubuntu-[INFO]:-Building primary segment instance array, please wait...
..
20220611:12:20:17:005363 gpinitsystem:8837967a36f4:ubuntu-[INFO]:-Checking Master host
20220611:12:20:17:005363 gpinitsystem:8837967a36f4:ubuntu-[INFO]:-Checking new segment hosts, please wait...
20220611:12:20:18:005363 gpinitsystem:8837967a36f4:ubuntu-[WARN]:-Host 8837967a36f4 open files limit is 1024 should be >= 65535
..
20220611:12:20:19:005363 gpinitsystem:8837967a36f4:ubuntu-[INFO]:-Checking new segment hosts, Completed
20220611:12:20:19:005363 gpinitsystem:8837967a36f4:ubuntu-[INFO]:-Greenplum Database Creation Parameters
20220611:12:20:19:005363 gpinitsystem:8837967a36f4:ubuntu-[INFO]:---------------------------------------
20220611:12:20:19:005363 gpinitsystem:8837967a36f4:ubuntu-[INFO]:-Master Configuration
20220611:12:20:19:005363 gpinitsystem:8837967a36f4:ubuntu-[INFO]:---------------------------------------
20220611:12:20:19:005363 gpinitsystem:8837967a36f4:ubuntu-[INFO]:-Master instance name       = GPDB SINGLENODE
20220611:12:20:19:005363 gpinitsystem:8837967a36f4:ubuntu-[INFO]:-Master hostname            = 8837967a36f4
20220611:12:20:19:005363 gpinitsystem:8837967a36f4:ubuntu-[INFO]:-Master port                = 5432
20220611:12:20:19:005363 gpinitsystem:8837967a36f4:ubuntu-[INFO]:-Master instance dir        = /home/ubuntu/gpdb/master/gpsne-1
20220611:12:20:19:005363 gpinitsystem:8837967a36f4:ubuntu-[INFO]:-Master LOCALE              = en_US.utf8
20220611:12:20:19:005363 gpinitsystem:8837967a36f4:ubuntu-[INFO]:-Greenplum segment prefix   = gpsne
20220611:12:20:19:005363 gpinitsystem:8837967a36f4:ubuntu-[INFO]:-Master Database            =
20220611:12:20:19:005363 gpinitsystem:8837967a36f4:ubuntu-[INFO]:-Master connections         = 250
20220611:12:20:19:005363 gpinitsystem:8837967a36f4:ubuntu-[INFO]:-Master buffers             = 128000kB
20220611:12:20:19:005363 gpinitsystem:8837967a36f4:ubuntu-[INFO]:-Segment connections        = 750
20220611:12:20:19:005363 gpinitsystem:8837967a36f4:ubuntu-[INFO]:-Segment buffers            = 128000kB
20220611:12:20:19:005363 gpinitsystem:8837967a36f4:ubuntu-[INFO]:-Checkpoint segments        = 8
20220611:12:20:19:005363 gpinitsystem:8837967a36f4:ubuntu-[INFO]:-Encoding                   = UNICODE
20220611:12:20:19:005363 gpinitsystem:8837967a36f4:ubuntu-[INFO]:-Postgres param file        = Off
20220611:12:20:19:005363 gpinitsystem:8837967a36f4:ubuntu-[INFO]:-Initdb to be used          = /opt/greenplum-db-6.20.5/bin/initdb
20220611:12:20:19:005363 gpinitsystem:8837967a36f4:ubuntu-[INFO]:-GP_LIBRARY_PATH is         = /opt/greenplum-db-6.20.5/lib
20220611:12:20:19:005363 gpinitsystem:8837967a36f4:ubuntu-[INFO]:-HEAP_CHECKSUM is           = on
20220611:12:20:19:005363 gpinitsystem:8837967a36f4:ubuntu-[INFO]:-HBA_HOSTNAMES is           = 0
20220611:12:20:19:005363 gpinitsystem:8837967a36f4:ubuntu-[WARN]:-Ulimit check               = Warnings generated, see log file <<<<<
20220611:12:20:19:005363 gpinitsystem:8837967a36f4:ubuntu-[INFO]:-Array host connect type    = Single hostname per node
20220611:12:20:19:005363 gpinitsystem:8837967a36f4:ubuntu-[INFO]:-Master IP address [1]      = 172.17.0.2
20220611:12:20:19:005363 gpinitsystem:8837967a36f4:ubuntu-[INFO]:-Standby Master             = Not Configured
20220611:12:20:19:005363 gpinitsystem:8837967a36f4:ubuntu-[INFO]:-Number of primary segments = 2
20220611:12:20:19:005363 gpinitsystem:8837967a36f4:ubuntu-[INFO]:-Total Database segments    = 2
20220611:12:20:19:005363 gpinitsystem:8837967a36f4:ubuntu-[INFO]:-Trusted shell              = ssh
20220611:12:20:19:005363 gpinitsystem:8837967a36f4:ubuntu-[INFO]:-Number segment hosts       = 1
20220611:12:20:19:005363 gpinitsystem:8837967a36f4:ubuntu-[INFO]:-Mirroring config           = OFF
20220611:12:20:19:005363 gpinitsystem:8837967a36f4:ubuntu-[INFO]:----------------------------------------
20220611:12:20:19:005363 gpinitsystem:8837967a36f4:ubuntu-[INFO]:-Greenplum Primary Segment Configuration
20220611:12:20:19:005363 gpinitsystem:8837967a36f4:ubuntu-[INFO]:----------------------------------------
20220611:12:20:19:005363 gpinitsystem:8837967a36f4:ubuntu-[INFO]:-8837967a36f4 	6000 	8837967a36f4 	/home/ubuntu/gpdb/primary/gpsne0 	2
20220611:12:20:19:005363 gpinitsystem:8837967a36f4:ubuntu-[INFO]:-8837967a36f4 	6001 	8837967a36f4 	/home/ubuntu/gpdb/primary/gpsne1 	3
```

输入“y”就可以继续安装了。

完成之后就可以开心的使用了。

### Ubuntu 20.04 或其他Linux版本

#### 下载源代码

```bash
git clone git@github.com:greenplum-db/gpdb.git
```

#### 安装依赖

在下载的源码目录下执行：

```bash
export DEBIAN_FRONTEND=noninteractive # 跳过测试用的一些参数
sudo ./README.Ubuntu.bash
```

#### 编译源码

```bash
# Configure build environment to install at /usr/local/gpdb
./configure --with-perl --with-python --with-libxml --with-gssapi --prefix=/usr/local/gpdb

# Compile and install
make -j3
make -j3 install

# Bring in greenplum environment into your running shell
source /usr/local/gpdb/greenplum_path.sh

# Start demo cluster
make create-demo-cluster
# (gpdemo-env.sh contains __PGPORT__ and __MASTER_DATA_DIRECTORY__ values)
source gpAux/gpdemo/gpdemo-env.sh
```

#### 修改参数

数据目录和TCP端口都可以在运行过程中修改，执行以下命令即可：

```bash
DATADIRS=/tmp/gpdb-cluster PORT_BASE=5555 make cluster
```

回归测试也可以在运行中修改参数：

```bash
PGPORT=5555 make installcheck-world
```

如果要关闭`GPORCA`，GP就会使用`Postgres Planner`来做查询优化。调整参数执行一下命令即可：

```
set optimizer=off;
```

#### 执行测试

```
make installcheck-world
```
