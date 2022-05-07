---
layout: post
title: 安装MySQL并设置lower_case_table_names参数
date: 2021-07-24
tags: ["MySQL", "Ubuntu"]
categories: MySQL
description: 安装MySQL并设置lower_case_table_names参数
---

 I can get it to work with a workaround. By re-initializing MySQL with the new value for lower_case_table_names after its installation. The following steps apply to a new installation. If you have already data in a database, export it first to import it back later:

## Install MySQL:

```bash
sudo apt-get update    
sudo apt-get install mysql-server -y
```

## Stop the MySQL service:
```bash
sudo service mysql stop
```

## Delete the MySQL data directory:
```bash
sudo rm -rf /var/lib/mysql
```

## Recreate the MySQL data directory (yes, it is not sufficient to just delete its content):
```bash
sudo mkdir /var/lib/mysql    
sudo chown mysql:mysql /var/lib/mysql
sudo chmod 700 /var/lib/mysql
```

## Add `lower_case_table_names = 1` to the `[mysqld]` section in `/etc/mysql/mysql.conf.d/mysqld.cnf`.

## Re-initialize MySQL with `--lower_case_table_names=1`:
```bash
sudo mysqld --defaults-file=/etc/mysql/my.cnf --initialize --lower_case_table_names=1 --user=mysql --console
```

## Start the MySQL service:
```bash
sudo service mysql start
```

## Retrieve the new generated password for MySQL user root:
```bash
sudo grep 'temporary password' /var/log/mysql/error.log
```

## Change the password of MySQL user root either by:
```bash
sudo mysql -u root -p
```
and executing:
```bash
ALTER USER 'root'@'localhost' IDENTIFIED BY 'MyNewPa$$w0rd';
```

afterwards, OR by calling the "hardening" script anyway:
```shell
sudo mysql_secure_installation
```

After that, you can verify the lower_case_table_names setting by entering the MySQL shell:

```bash
sudo mysql -u root -p
```

and executing:

```sql
SHOW VARIABLES LIKE 'lower_case_%';
```
Expected output:

```
+------------------------+-------+
| Variable_name          | Value |
+------------------------+-------+
| lower_case_file_system | OFF   |
| lower_case_table_names | 1     |
+------------------------+-------+
```
