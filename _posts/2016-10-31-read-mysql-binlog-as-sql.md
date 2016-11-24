---
layout: post
title: 以SQL方式读取MySQL的GTID格式Binlog
tags:
- MySQL
- Binlog
- GTID
categories: 数据库
description: 以SQL方式读取MySQL的GTID格式Binlog
---

将GTID格式的binlog转换为SQL的命令：

> ```mysqlbinlog --no-defaults -v --base64-output=DECODE-ROWS  --skip-gtids mysql-bin.000714```

--------

下面讲详细介绍每个参数的意义，以及为什么需要添加这些参数。

## MySQL及mysqlbinlog的版本

## mysqlbinlog

`mysqlbinlog`是MySQL官方提供的读取binlog的工具。其执行方法如下：
```bash
mysqlbinlog [options] log_file ...
```

### `--no-defaults`参数

直接执行`mysqlbinlog`命令如下：
```bash
mysqlbinlog mysql-bin.000858
```
执行失败，报如下错误：
```
mysqlbinlog: unknown variable 'default-character-set=utf8mb4'
```
因为使用了默认编码，但是mysqlbinlog又无法正常解析。增加`--no-defaults`参数直接跳过默认编码格式。

手册中关于该参数的说明：
> **--no-defaults**

> Do not read any option files. If program startup fails due to reading unknown options from an option file, --no-defaults can be used to prevent them from being read.

> The exception is that the .mylogin.cnf file, if it exists, is read in all cases. This permits passwords to be specified in a safer way than on the command line even when --no-defaults is used. (.mylogin.cnf is created by the mysql_config_editor utility. See mysql_config_editor(1).)

### -v参数

增加`--no-defaults`参数，执行以下命令：
```bash
mysqlbinlog --no-defaults mysql-bin.000858
```
输出如下：
```
/*!50530 SET @@SESSION.PSEUDO_SLAVE_MODE=1*/;
/*!40019 SET @@session.max_insert_delayed_threads=0*/;
/*!50003 SET @OLD_COMPLETION_TYPE=@@COMPLETION_TYPE,COMPLETION_TYPE=0*/;
DELIMITER /*!*/;
# at 4
#161123  4:17:00 server id 1  end_log_pos 120 CRC32 0x095b60fa  Start: binlog v 4, server v 5.6.27-0ubuntu0.14.04.1-log created 161123  4:17:00
BINLOG '
vKc0WA8BAAAAdAAAAHgAAAAAAAQANS42LjI3LTB1YnVudHUwLjE0LjA0LjEtbG9nAAAAAAAAAAAA
AAAAAAAAAAAAAAAAAAAAAAAAEzgNAAgAEgAEBAQEEgAAXAAEGggAAAAICAgCAAAACgoKGRkAAfpg
Wwk=
'/*!*/;
# at 120
#161123  4:17:00 server id 1  end_log_pos 231 CRC32 0xbbec3fba  Previous-GTIDs
# 9dc1a422-aeb6-11e5-80cd-00163e001e5d:1-5846827,
# fd076ada-69cf-11e6-84d8-00163e0c1dbe:1-381819
# at 231
#161123  4:17:00 server id 1  end_log_pos 279 CRC32 0x3f0812c4  GTID [commit=yes]
SET @@SESSION.GTID_NEXT= '9dc1a422-aeb6-11e5-80cd-00163e001e5d:5846828'/*!*/;
# at 279
#161123  4:17:00 server id 1  end_log_pos 353 CRC32 0x6982c05a  Query   thread_id=2662386       exec_time=0     error_code=0
SET TIMESTAMP=1479845820/*!*/;
SET @@session.pseudo_thread_id=2662386/*!*/;
SET @@session.foreign_key_checks=1, @@session.sql_auto_is_null=0, @@session.unique_checks=1, @@session.autocommit=1/*!*/;
SET @@session.sql_mode=1075838976/*!*/;
SET @@session.auto_increment_increment=1, @@session.auto_increment_offset=1/*!*/;
/*!\C utf8mb4 *//*!*/;
SET @@session.character_set_client=45,@@session.collation_connection=45,@@session.collation_server=224/*!*/;
SET @@session.lc_time_names=0/*!*/;
SET @@session.collation_database=DEFAULT/*!*/;
BEGIN
/*!*/;
# at 353
#161123  4:17:00 server id 1  end_log_pos 450 CRC32 0xb1d11f47  Table_map: `linahr`.`qrtz_fired_triggers` mapped to number 877
# at 450
#161123  4:17:00 server id 1  end_log_pos 825 CRC32 0x0d1392a7  Update_rows: table id 877 flags: STMT_END_F

BINLOG '
vKc0WBMBAAAAYQAAAMIBAAAAAG0DAAAAAAEABmxpbmFocgATcXJ0el9maXJlZF90cmlnZ2VycwAN
Dw8PDw8ICAMPDw8PDxRoAR0BWAJYAlgCMABYAlgCAwADAAAeRx/RsQ==
vKc0WB8BAAAAdwEAADkDAAAAAG0DAAAAAAEAAgAN/////wDmDABDUk1zY2hlZHVsZXImADEyYzZl
ZWNkMDgwZTE0Nzc1ODMzMjQ0ODYxNDc3NTgzMzc2NjQyGABxY2xvdWRWaWRlb1VwbG9hZFRyaWdn
ZXIHAERFRkFVTFQZADEyYzZlZWNkMDgwZTE0Nzc1ODMzMjQ0ODZTEa+NWAEAAGA2r41YAQAAAAAA
```
其中并没有出现我们想要的SQL语句。这时候就需要增加`-v`参数了。

`-v`参数在手册中的描述如下：
> --verbose, -v
>
> Reconstruct row events and display them as commented SQL statements. If this option is given twice, the output includes comments to indicate column data types and some metadata.
> 
> For examples that show the effect of --base64-output and --verbose on row event output, see the section called “MYSQLBINLOG ROW EVENT DISPLAY”.

增加`-v`参数，可以看到如下的输出：
```
BINLOG '
vKc0WBMBAAAAXQAAAJYDAAAAAHQDAAAAAAMABmxpbmFocgANcXJ0el90cmlnZ2VycwAQDw8PDw8P
CAgDDw8ICA8C/BNoAVgCWAJYAlgC7gIwABgAWAIC4PGAPOuQ
vKc0WB8BAAAAOwEAANEEAAAAAHQDAAAAAAEAAgAQ/////yAgDABDUk1zY2hlZHVsZXIYAHFjbG91
ZFZpZGVvVXBsb2FkVHJpZ2dlcgcAREVGQVVMVBcAcWNsb3VkVmlkZW9VcGxvYWREZXRhaWwHAERF
RkFVTFRgNq+NWAEAAABMro1YAQAAAAAAAAhBQ1FVSVJFRARDUk9OeD/UBlgBAAAAAAAAAAAAAAAA
AAAgIAwAQ1JNc2NoZWR1bGVyGABxY2xvdWRWaWRlb1VwbG9hZFRyaWdnZXIHAERFRkFVTFQXAHFj
bG91ZFZpZGVvVXBsb2FkRGV0YWlsBwBERUZBVUxUwCCwjVgBAABgNq+NWAEAAAAAAAAHV0FJVElO
RwRDUk9OeD/UBlgBAAAAAAAAAAAAAAAAAAAIicmV
'/*!*/;
### UPDATE `linahr`.`qrtz_triggers`
### WHERE
###   @1='CRMscheduler'
###   @2='qcloudVideoUploadTrigger'
###   @3='DEFAULT'
###   @4='qcloudVideoUploadDetail'
###   @5='DEFAULT'
###   @6=NULL
###   @7=1479845820000
###   @8=1479845760000
###   @9=0
###   @10='ACQUIRED'
###   @11='CRON'
###   @12=1477583323000
###   @13=0
###   @14=NULL
###   @15=0
###   @16=''
### SET
###   @1='CRMscheduler'
###   @2='qcloudVideoUploadTrigger'
###   @3='DEFAULT'
###   @4='qcloudVideoUploadDetail'
###   @5='DEFAULT'
###   @6=NULL
###   @7=1479845880000
###   @8=1479845820000
###   @9=0
###   @10='WAITING'
###   @11='CRON'
###   @12=1477583323000
###   @13=0
###   @14=NULL
###   @15=0
###   @16=''
# at 1233
#161123  4:17:00 server id 1  end_log_pos 1264 CRC32 0x484eacf5         Xid = 447774161
COMMIT/*!*/;
# at 1264
#161123  4:17:00 server id 1  end_log_pos 1312 CRC32 0xc364953c         GTID [commit=yes]
SET @@SESSION.GTID_NEXT= '9dc1a422-aeb6-11e5-80cd-00163e001e5d:5846829'/*!*/;
# at 1312
#161123  4:17:00 server id 1  end_log_pos 1386 CRC32 0xbf84bd5c         Query   thread_id=2662325       exec_time=0     error_code=0
SET TIMESTAMP=1479845820/*!*/;
BEGIN
/*!*/;
# at 1386
#161123  4:17:00 server id 1  end_log_pos 1483 CRC32 0xb85e9cff         Table_map: `linahr`.`qrtz_fired_triggers` mapped to number 877
# at 1483
#161123  4:17:00 server id 1  end_log_pos 1691 CRC32 0x1447d61d         Delete_rows: table id 877 flags: STMT_END_F
```
增加`-vv`参数，还可以输出一些额外的数据类型信息。如下：
```
BINLOG '
vKc0WBMBAAAAYQAAAMIBAAAAAG0DAAAAAAEABmxpbmFocgATcXJ0el9maXJlZF90cmlnZ2VycwAN
Dw8PDw8ICAMPDw8PDxRoAR0BWAJYAlgCMABYAlgCAwADAAAeRx/RsQ==
vKc0WB8BAAAAdwEAADkDAAAAAG0DAAAAAAEAAgAN/////wDmDABDUk1zY2hlZHVsZXImADEyYzZl
ZWNkMDgwZTE0Nzc1ODMzMjQ0ODYxNDc3NTgzMzc2NjQyGABxY2xvdWRWaWRlb1VwbG9hZFRyaWdn
ZXIHAERFRkFVTFQZADEyYzZlZWNkMDgwZTE0Nzc1ODMzMjQ0ODZTEa+NWAEAAGA2r41YAQAAAAAA
AAhBQ1FVSVJFRAEwATAA4AwAQ1JNc2NoZWR1bGVyJgAxMmM2ZWVjZDA4MGUxNDc3NTgzMzI0NDg2
MTQ3NzU4MzM3NjY0MhgAcWNsb3VkVmlkZW9VcGxvYWRUcmlnZ2VyBwBERUZBVUxUGQAxMmM2ZWVj
ZDA4MGUxNDc3NTgzMzI0NDg2dTavjVgBAABgNq+NWAEAAAAAAAAJRVhFQ1VUSU5HFwBxY2xvdWRW
aWRlb1VwbG9hZERldGFpbAcAREVGQVVMVAEwATCnkhMN
'/*!*/;
### UPDATE `linahr`.`qrtz_fired_triggers`
### WHERE
###   @1='CRMscheduler' /* VARSTRING(360) meta=360 nullable=0 is_null=0 */
###   @2='12c6eecd080e14775833244861477583376642' /* VARSTRING(285) meta=285 nullable=0 is_null=0 */
###   @3='qcloudVideoUploadTrigger' /* VARSTRING(600) meta=600 nullable=0 is_null=0 */
###   @4='DEFAULT' /* VARSTRING(600) meta=600 nullable=0 is_null=0 */
###   @5='12c6eecd080e1477583324486' /* VARSTRING(600) meta=600 nullable=0 is_null=0 */
###   @6=1479845810515 /* LONGINT meta=0 nullable=0 is_null=0 */
###   @7=1479845820000 /* LONGINT meta=0 nullable=0 is_null=0 */
###   @8=0 /* INT meta=0 nullable=0 is_null=0 */
###   @9='ACQUIRED' /* VARSTRING(48) meta=48 nullable=0 is_null=0 */
###   @10=NULL /* VARSTRING(48) meta=600 nullable=1 is_null=1 */
###   @11=NULL /* VARSTRING(48) meta=600 nullable=1 is_null=1 */
###   @12='0' /* VARSTRING(3) meta=3 nullable=1 is_null=0 */
###   @13='0' /* VARSTRING(3) meta=3 nullable=1 is_null=0 */
### SET
###   @1='CRMscheduler' /* VARSTRING(360) meta=360 nullable=0 is_null=0 */
###   @2='12c6eecd080e14775833244861477583376642' /* VARSTRING(285) meta=285 nullable=0 is_null=0 */
###   @3='qcloudVideoUploadTrigger' /* VARSTRING(600) meta=600 nullable=0 is_null=0 */
###   @4='DEFAULT' /* VARSTRING(600) meta=600 nullable=0 is_null=0 */
###   @5='12c6eecd080e1477583324486' /* VARSTRING(600) meta=600 nullable=0 is_null=0 */
###   @6=1479845820021 /* LONGINT meta=0 nullable=0 is_null=0 */
###   @7=1479845820000 /* LONGINT meta=0 nullable=0 is_null=0 */
###   @8=0 /* INT meta=0 nullable=0 is_null=0 */
###   @9='EXECUTING' /* VARSTRING(48) meta=48 nullable=0 is_null=0 */
###   @10='qcloudVideoUploadDetail' /* VARSTRING(600) meta=600 nullable=1 is_null=0 */
###   @11='DEFAULT' /* VARSTRING(600) meta=600 nullable=1 is_null=0 */
###   @12='0' /* VARSTRING(3) meta=3 nullable=1 is_null=0 */
###   @13='0' /* VARSTRING(3) meta=3 nullable=1 is_null=0 */
# at 825
#161123  4:17:00 server id 1  end_log_pos 918 CRC32 0x90eb3c80  Table_map: `linahr`.`qrtz_triggers` mapped to number 884
# at 918
#161123  4:17:00 server id 1  end_log_pos 1233 CRC32 0x95c98908         Update_rows: table id 884 flags: STMT_END_F
```

### `--base64-output=DECODE-ROWS`参数

`--base64-decode`参数在手册上的说明如下：
> --base64-output=value
> 
> This option determines when events should be displayed encoded as base-64 strings using BINLOG statements. The option has these permissible values (not case sensitive):

> ·   AUTO ("automatic") or UNSPEC ("unspecified") displays BINLOG statements automatically when necessary (that is, for format description events and row events). If no --base64-output option is given, the effect is the same as --base64-output=AUTO.

>        Note
>        Automatic BINLOG display is the only safe behavior if you intend to use the output of mysqlbinlog to re-execute binary log file contents. The other option values are intended only for debugging or testing purposes because they may produce output that does not include all events in executable form.

> ·   NEVER causes BINLOG statements not to be displayed.  mysqlbinlog exits with an error if a row event is found that must be displayed using BINLOG.

> ·   DECODE-ROWS specifies to mysqlbinlog that you intend for row events to be decoded and displayed as commented SQL statements by also specifying the --verbose option. Like NEVER, DECODE-ROWS suppresses display of BINLOG statements, but unlike NEVER, it does not exit with an error if a row event is found.

> For examples that show the effect of --base64-output and --verbose on row event output, see the section called “MYSQLBINLOG ROW EVENT DISPLAY”.

增加`--base64-output=DECODE-ROWS`参数执行：
```
mysqlbinlog --no-defaults -v --base64-output=DECODE-ROWS mysql-bin.000858 > tmp.txt
```
输出如下，可以看到`BINLOG '`开头，以base64编码的数据没有了：
```
SET TIMESTAMP=1479845826/*!*/;
BEGIN
/*!*/;
# at 2697
#161123  4:17:06 server id 1  end_log_pos 2773 CRC32 0x3b984e8d         Table_map: `linahr_pay`.`qrtz_scheduler_state` mapped to number 1180
# at 2773
#161123  4:17:06 server id 1  end_log_pos 2925 CRC32 0x394f0f8b         Update_rows: table id 1180 flags: STMT_END_F
### UPDATE `linahr_pay`.`qrtz_scheduler_state`
### WHERE
###   @1='CRMscheduler'
###   @2='f5589e5e5a731477582086079'
###   @3=1479845806492
###   @4=20000
### SET
###   @1='CRMscheduler'
###   @2='f5589e5e5a731477582086079'
###   @3=1479845826494
###   @4=20000
# at 2925
#161123  4:17:06 server id 1  end_log_pos 2956 CRC32 0x7d1d1eab         Xid = 447774290
COMMIT/*!*/;
```

### `--skip-gtids`参数

> --skip-gtids[=(true|false)]

> Do not display any GTIDs in the output. This is needed when writing to a dump file from one or more binary logs containing GTIDs, as shown in this example:

>		shell> mysqlbinlog --skip-gtids binlog.000001 >  /tmp/dump.sql
>		shell> mysqlbinlog --skip-gtids binlog.000002 >> /tmp/dump.sql
>		shell> mysql -u root -p -e "source /tmp/dump.sql"

> The use of this option is otherwise not normally recommended in production.

## 最终命令
```bash
mysqlbinlog --no-defaults -v --base64-output=DECODE-ROWS --skip-gtids mysql-bin.000858
```