---
layout: post
title: MySQL服务器配置
tags:
- MySQL
categories: 数据库
description: MySQL服务器配置，包括基本配置和晋级配置
---

MySQL版本信息：
> mysqld  Ver 5.6.31-0ubuntu0.14.04.2 for debian-linux-gnu on x86_64 ((Ubuntu))

## 基本配置

```diff
#
# The MySQL database server configuration file.
#
# You can copy this to one of:
# - "/etc/mysql/my.cnf" to set global options,
# - "~/.my.cnf" to set user-specific options.
# 
# One can use all long options that the program supports.
# Run program with --help to get a list of available options and with
# --print-defaults to see which it would actually understand and use.
#
# For explanations see
# http://dev.mysql.com/doc/mysql/en/server-system-variables.html

# This will be passed to all mysql clients
# It has been reported that passwords should be enclosed with ticks/quotes
# escpecially if they contain "#" chars...
# Remember to edit /etc/mysql/debian.cnf when changing the socket location.
[client]
port            = 3306
socket          = /var/run/mysqld/mysqld.sock
+ default-character-set = utf8mb4

# Here is entries for some specific programs
# The following values assume you have at least 32M ram

# This was formally known as [safe_mysqld]. Both versions are currently parsed.
[mysqld_safe]
socket          = /var/run/mysqld/mysqld.sock
nice            = 0

[mysqld]
#
# * Basic Settings
#
+ skip-host-cache
user            = mysql
pid-file        = /var/run/mysqld/mysqld.pid
socket          = /var/run/mysqld/mysqld.sock
port            = 3306
basedir         = /usr
datadir         = /var/lib/mysql
tmpdir          = /tmp
lc-messages-dir = /usr/share/mysql
+ explicit_defaults_for_timestamp
skip-external-locking
#
# Instead of skip-networking the default is now to listen only on
# localhost which is more compatible and is not less secure.
- bind-address           = 127.0.0.1
+ #bind-address           = 127.0.0.1
#
# * Fine Tuning
#
key_buffer              = 16M
max_allowed_packet      = 16M
thread_stack            = 192K
thread_cache_size       = 8
# This replaces the startup script and checks MyISAM tables if needed
# the first time they are touched
myisam-recover         = BACKUP
- #max_connections        = 100
+ max_connections        = 1000
#table_cache            = 64
thread_concurrency     = 10
#
# * Query Cache Configuration
#
query_cache_limit       = 1M
query_cache_size        = 16M
+ skip_name_resolve       = 1
+ lower_case_table_names  = 1
+ max_allowed_packet      = 32M
#
# * Logging and Replication
#
# Both location gets rotated by the cronjob.
# Be aware that this log type is a performance killer.
# As of 5.1 you can enable the log at runtime!
#general_log_file        = /var/log/mysql/mysql.log
#general_log             = 1
#
# Error log - should be very few entries.
#
log_error = /var/log/mysql/error.log
#
# Here you can see queries with especially long duration
#log_slow_queries       = /var/log/mysql/mysql-slow.log
#long_query_time = 2
#log-queries-not-using-indexes
+ # 千万不要使用log_slow_queries，这是一个错误的参数，会导致MySQL无法启动
+ slow_query_log          = 1
+ slow_query_log_file     = /var/log/mysql/slow.log
+ long_query_time         = 10
#
# The following can be used as easy to replay backup logs or for replication.
# note: if you are setting up a replication slave, see README.Debian about
#       other settings you may need to change.
- #server-id              = 1
+ server-id               = 2
- #log_bin                 = /var/log/mysql/mysql-bin.log
+ log_bin                 = /var/log/mysql/mysql-bin.log
expire_logs_days        = 10
max_binlog_size         = 100M
+ read-only               = 0
#binlog_do_db           = include_database_name
#binlog_ignore_db       = include_database_name
+ binlog-ignore-db        = mysql
+ binlog-ignore-db        = information_schema
+ binlog-ignore-db        = performance_schema
+ ## GTID: Global Transaction ID
+ gtid_mode               = on
+ enforce_gtid_consistency= on
+ log-slave-updates       = 1
+ # For all slave servers
+ skip_slave_start        = 1
#
# * InnoDB
#
# InnoDB is enabled by default with a 10MB datafile in /var/lib/mysql/.
# Read the manual for more InnoDB related options. There are many!
#
# * Security Features
#
# Read the manual, too, if you want chroot!
# chroot = /var/lib/mysql/
#
# For generating SSL certificates I recommend the OpenSSL GUI "tinyca".
#
# ssl-ca=/etc/mysql/cacert.pem
# ssl-cert=/etc/mysql/server-cert.pem
# ssl-key=/etc/mysql/server-key.pem

+ max_connect_errors      = 20
+ interactive_timeout     = 172800
+ wait_timeout            = 172800

# Character sets
+ character-set-client-handshake = FALSE
+ character-set-server    = utf8mb4
+ collation-server        = utf8mb4_unicode_ci

[mysqldump]
quick
quote-names
+ max_allowed_packet      = 32M

[mysql]
#no-auto-rehash # faster start of mysql but no tab completition
+ default-character-set   = utf8mb4

[isamchk]
key_buffer              = 16M

#
# * IMPORTANT: Additional settings that can override those from this file!
#   The files must end with '.cnf', otherwise they'll be ignored.
#
!includedir /etc/mysql/conf.d/
```
