---
layout: post
title: 使用GTID机制创建MySQL主从备份
tags:
- MySQL
- GTID
- 主从复制
categories: 数据库
description: 使用GTID机制创建MySQL主从备份
---

> **注意：**GTID机制只适用与MySQL 5.6.10及更新版本。

## GTID比传统复制的优势

1. 更简单的实现failover，不用以前那样在需要找log_file和log_Pos。
2. 更简单的搭建主从复制。
3. 比传统复制更加安全。
4. GTID是连续没有空洞的，因此主从库出现数据冲突时，可以用添加空事物的方式进行跳过。

## GTID的工作原理：

1. master更新数据时，会在事务前产生GTID，一同记录到binlog日志中。
2. slave端的i/o 线程将变更的binlog，写入到本地的relay log中。
3. sql线程从relay log中获取GTID，然后对比slave端的binlog是否有记录。
4. 如果有记录，说明该GTID的事务已经执行，slave会忽略。
5. 如果没有记录，slave就会从relay log中执行该GTID的事务，并记录到binlog。
6. 在解析过程中会判断是否有主键，如果没有就用二级索引，如果没有就用全部扫描。

**要点：**

1. slave在接受master的binlog时，会校验master的GTID是否已经执行过（一个服务器只能执行一次）。
2. 为了保证主从数据的一致性，多线程只能同时执行一个GTID。

## MySQL启动GTID机制

主从MySQL的配置基本一致，区别在于`server_id`**必须**不同：
```ini
[mysqld]
server-id               = 2
# 在GTID机制下从服务器也必须开启
log_bin                 = /var/log/mysql/mysql-bin.log
# 参考下文说明，强烈推荐使用row
binlog_format           = row
expire_logs_days        = 10
max_binlog_size         = 100M
read-only               = 0
binlog-ignore-db        = mysql
binlog-ignore-db        = information_schema
binlog-ignore-db        = performance_schema
#binlog_ignore_db       = include_database_name

## GTID: Global Transaction ID
gtid_mode               = on
# 使用主从复制的时候必须启用
enforce_gtid_consistency= on
# 使用主从复制的时候必须启用
log-slave-updates       = 1
# For all slave servers
skip_slave_start        = 1
```
关于`binlog-format = row`的[官方描述](http://dev.mysql.com/doc/refman/5.7/en/binary-log-setting.html)：
> **Warning**
> When using statement-based logging for replication, it is possible for the data on the master and slave to become different if a statement is designed in such a way that the data modification is nondeterministic; that is, it is left to the will of the query optimizer. In general, this is not a good practice even outside of replication. For a detailed explanation of this issue, see Section B.5.7, “Known Issues in MySQL”.

重新启动MySQL服务器，可以查看GTID是否正常启动：
```
mysql> show global variables like '%gtid%';
+---------------------------------+-----------------------------------------------+
| Variable_name                   | Value                                         |
+---------------------------------+-----------------------------------------------+
| binlog_gtid_simple_recovery     | OFF                                           |
| enforce_gtid_consistency        | ON                                            |
| gtid_executed                   | 3591a291-699c-11e6-8386-0242ac1100f2:1-1830   |
| gtid_mode                       | ON                                            |
| gtid_owned                      | 3591a291-699c-11e6-8386-0242ac1100f2:1831#127 |
| gtid_purged                     |                                               |
| simplified_binlog_gtid_recovery | OFF                                           |
+---------------------------------+-----------------------------------------------+
7 rows in set (0.02 sec)
```

执行一条更新语句，然后就可以看到GTID的状态了：
```
mysql> show master status\G;
*************************** 1. row ***************************
             File: mysql-bin.000001
         Position: 1070635555
     Binlog_Do_DB: 
 Binlog_Ignore_DB: mysql,information_schema,performance_schema
Executed_Gtid_Set: 3591a291-699c-11e6-8386-0242ac1100f2:1-1745
1 row in set (0.00 sec)

ERROR: 
No query specified
```

## 启动slave

### 创建Master的Replica帐号：
```sql
CREATE USER 'repl'@'%' IDENTIFIED BY 'slave';
GRANT REPLICATION SLAVE ON *.* TO 'repl'@'%';
```

### 在新系统中启动Slave

如果所有的Binlog都存在Master中，我们认为这是一个新的MySQL系统，此时可以非常容易的启动Slave，只需要在Slave中执行：
```sql
stop slave;
change master to master_host='192.168.1.3', master_port=3307, master_user='repl', master_password='slave', master_auto_position=1;
start slave;
```
最后一个参数`master_auto_position=1`告诉Slave服务器自动去寻找Binlog的位置，并且启动Slave。

此时可以查看Slave的状态：
```
mysql> show slave status\G;
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: 192.168.1.3
                  Master_User: repl
                  Master_Port: 3307
                Connect_Retry: 60
              Master_Log_File: mysql-bin.000005
          Read_Master_Log_Pos: 22069
               Relay_Log_File: mysqld-relay-bin.000005
                Relay_Log_Pos: 29521031
        Relay_Master_Log_File: mysql-bin.000003
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
              Replicate_Do_DB: 
          Replicate_Ignore_DB: 
           Replicate_Do_Table: 
       Replicate_Ignore_Table: 
      Replicate_Wild_Do_Table: 
  Replicate_Wild_Ignore_Table: 
                   Last_Errno: 0
                   Last_Error: 
                 Skip_Counter: 0
          Exec_Master_Log_Pos: 29520821
              Relay_Log_Space: 973528215
              Until_Condition: None
               Until_Log_File: 
                Until_Log_Pos: 0
           Master_SSL_Allowed: No
           Master_SSL_CA_File: 
           Master_SSL_CA_Path: 
              Master_SSL_Cert: 
            Master_SSL_Cipher: 
               Master_SSL_Key: 
        Seconds_Behind_Master: 6454
Master_SSL_Verify_Server_Cert: No
                Last_IO_Errno: 0
                Last_IO_Error: 
               Last_SQL_Errno: 0
               Last_SQL_Error: 
  Replicate_Ignore_Server_Ids: 
             Master_Server_Id: 1
                  Master_UUID: 3591a291-699c-11e6-8386-0242ac1100f2
             Master_Info_File: /var/lib/mysql/master.info
                    SQL_Delay: 0
          SQL_Remaining_Delay: NULL
      Slave_SQL_Running_State: executing
           Master_Retry_Count: 86400
                  Master_Bind: 
      Last_IO_Error_Timestamp: 
     Last_SQL_Error_Timestamp: 
               Master_SSL_Crl: 
           Master_SSL_Crlpath: 
           Retrieved_Gtid_Set: 3591a291-699c-11e6-8386-0242ac1100f2:1-2114
            Executed_Gtid_Set: 3591a291-699c-11e6-8386-0242ac1100f2:1-1750
                Auto_Position: 1
1 row in set (0.00 sec)

ERROR: 
No query specified
```
需要注意的是以下两个字段都应该为Yes：
```
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
```

### 在已有系统中启动Slave

使用mysqldump的方式：

1. 在备份的时候指定--master-data=2（来保存binlog的文件号和位置的命令）。
2. 使用mysqldump的命令在dump文件里可以看到下面两个信息：
```
SET @@SESSION.SQL_LOG_BIN=0;
SET @@GLOBAL.GTID_PURGED='7800a22c-95ae-11e4-983d-080027de205a:1-8';
```
3. 将备份还原到slave后，使用change master to命令挂载master端。

