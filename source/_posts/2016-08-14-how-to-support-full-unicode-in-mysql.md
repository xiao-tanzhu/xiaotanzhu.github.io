---
layout: post
title: How to support full Unicode in MySQL databases
date: 2016-08-14
tags:
- MySQL
- utf8mb4
- Emoji
categories: 数据库
description: How to support full Unicode in MySQL databases
---

如果你完全从头建立一个数据库，那么事情简单了很多，直接参考：[创建支持emoji表情的MySQL数据库（utf8mb4）](/2016/08/14/create-mysql-database-with-utf8mb4.html)。

如果你的数据库已经在投入使用，那么可以参考文章的以下部分。

> **Reference:** https://mathiasbynens.be/notes/mysql-utf8mb4

## UTF-8

[The UTF-8 encoding](https://encoding.spec.whatwg.org/#utf-8) can represent every symbol in the Unicode character set, which ranges from `U+000000` to `U+10FFFF`. That’s 1,114,112 possible symbols. (Not all of these Unicode code points have been assigned characters yet, but that doesn’t stop UTF-8 from being able to encode them.)

UTF-8 is a variable-width encoding; it encodes each symbol using **one to four 8-bit bytes**. Symbols with lower numerical code point values are encoded using fewer bytes. This way, UTF-8 is optimized for the common case where ASCII characters and other [BMP symbols](https://mathiasbynens.be/notes/javascript-encoding#bmp) (whose code points range from `U+000000` to `U+00FFFF`) are used — while still allowing astral symbols (whose code points range from `U+010000` to `U+10FFFF`) to be stored.

## MySQL’s `utf8`

MySQL’s utf8 charset only partially implements proper UTF-8 encoding. It can only store UTF-8-encoded symbols that consist of one to three bytes; encoded symbols that take up four bytes aren’t supported.

Since astral symbols (whose code points range from U+010000 to U+10FFFF) each consist of four bytes in UTF-8, you cannot store them using MySQL’s utf8 implementation. And if you are doing so, you'll get the following warning:

```
mysql> SHOW WARNINGS;
+---------+------+------------------------------------------------------------------------------+
| Level   | Code | Message                                                                      |
+---------+------+------------------------------------------------------------------------------+
| Warning | 1366 | Incorrect string value: '\xF0\x9D\x8C\x86' for column 'column_name' at row 1 |
+---------+------+------------------------------------------------------------------------------+
1 row in set (0.00 sec)
```

this behavior can lead to data loss, but it gets worse — it can result in security vulnerabilities. Here are some examples, all of which were discovered after publishing this write-up:

1. [PHP object injection vulnerability in WordPress < 3.6.1](https://tom.vg/2013/09/wordpress-php-object-injection/), leading to [remote code execution in combination with certain WordPress plugins](https://tom.vg/2013/12/wordpress-rce-exploit/)
2. [Email authentication bypass in Phabricator](https://hackerone.com/reports/2233)
3. [Stored XSS in WordPress 4.1.2](https://cedricvb.be/post/wordpress-stored-xss-vulnerability-4-1-2/)
4. [Remote command execution in the Joomla! CMS](https://www.reddit.com/r/netsec/comments/3wt0yk/critical_0day_remote_command_execution/cxz2qtc)

MySQL’s `utf8` encoding is awkwardly named, as it’s different from proper UTF-8 encoding. It doesn’t offer full Unicode support, which can lead to data loss or security vulnerabilities.

## MySQL’s `utf8mb4`

Luckily, [MySQL 5.5.3 (released in early 2010)](https://dev.mysql.com/doc/relnotes/mysql/5.5/en/news-5-5-3.html) introduced a new encoding called utf8mb4 which maps to proper UTF-8 and thus fully supports Unicode, including astral symbols.

### Switching from MySQL’s `utf8` to `utf8mb4`

**Step 1: Create a backup**

Create a backup of all the databases on the server you want to upgrade. Safety first!

**Step 2: Upgrade the MySQL server**

[Upgrade the MySQL server to v5.5.3+](https://dev.mysql.com/downloads/mysql/), or ask your server administrator to do it for you.

**Step 3: Modify databases, tables, and columns**

Change the character set and collation properties of the databases, tables, and columns to use `utf8mb4` instead of `utf8`.

```sql
# For each database:
ALTER DATABASE database_name CHARACTER SET = utf8mb4 COLLATE = utf8mb4_unicode_ci;
# For each table:
ALTER TABLE table_name CONVERT TO CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
# For each column:
ALTER TABLE table_name CHANGE column_name column_name VARCHAR(191) CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
# (Don’t blindly copy-paste this! The exact statement depends on the column type, maximum length, and other properties. The above line is just an example for a `VARCHAR` column.)
```
Since `utf8mb4` is fully backwards compatible with `utf8`, no mojibake or other forms of data loss should occur. (But you have a backup, right?)

**Step 4: Check the maximum length of columns and index keys**

This is probably the most tedious part of the whole upgrading process.

When converting from `utf8` to `utf8mb4`, the maximum length of a column or index key is unchanged in terms of bytes. Therefore, it is smaller in terms of characters, because the maximum length of a character is now four bytes instead of three.

For example, a `TINYTEXT` column can hold up to 255 bytes, which correlates to 85 three-byte or 63 four-byte characters. Let’s say you have a `TINYTEXT` column that uses `utf8` but must be able to contain more than 63 characters. Given this requirement, you can’t convert this column to `utf8mb4` unless you also change the data type to a longer type such as `TEXT` — because if you’d try to fill it with four-byte characters, you’d only be able to enter 63 characters, but not more.

The same goes for index keys. The InnoDB storage engine has a maximum index length of 767 bytes, so for `utf8` or `utf8mb4` columns, you can index a maximum of 255 or 191 characters, respectively. If you currently have `utf8` columns with indexes longer than 191 characters, you will need to index a smaller number of characters when using `utf8mb4`. (Because of this, I had to change some indexed VARCHAR(255) columns to VARCHAR(191).)

**Step 5: Modify connection, client, and server character sets**

In your application code, set the connection character set to utf8mb4. This can be done by simply replacing any variants of `SET NAMES utf8` with `SET NAMES utf8mb4`. If your old `SET NAMES` statement specified the collation, make sure to change that as well, e.g. `SET NAMES utf8 COLLATE utf8_unicode_ci` becomes `SET NAMES utf8mb4 COLLATE utf8mb4_unicode_ci`.

Make sure to set the client and server character set as well. I have the following in my MySQL configuration file (/etc/my.cnf):

```ini
[client]
default-character-set = utf8mb4

[mysql]
default-character-set = utf8mb4

[mysqld]
character-set-client-handshake = FALSE
character-set-server = utf8mb4
collation-server = utf8mb4_unicode_ci
```

You can easily confirm these settings work correctly:

```
mysql> SHOW VARIABLES WHERE Variable_name LIKE 'character\_set\_%' OR Variable_name LIKE 'collation%';
+--------------------------+--------------------+
| Variable_name            | Value              |
+--------------------------+--------------------+
| character_set_client     | utf8mb4            |
| character_set_connection | utf8mb4            |
| character_set_database   | utf8mb4            |
| character_set_filesystem | binary             |
| character_set_results    | utf8mb4            |
| character_set_server     | utf8mb4            |
| character_set_system     | utf8               |
| collation_connection     | utf8mb4_unicode_ci |
| collation_database       | utf8mb4_unicode_ci |
| collation_server         | utf8mb4_unicode_ci |
+--------------------------+--------------------+
10 rows in set (0.00 sec)
```

As you can see, all the relevant options are set to utf8mb4, except for `character_set_filesystem` which should be binary unless you’re on a file system that supports multi-byte UTF-8-encoded characters in file names, and `character_set_system` which is always utf8 and can’t be overridden.

**Step 6: Repair and optimize all tables**

You only need this when you meet some wired bugs. The details are in the reference: [How to support full Unicode in MySQL databases](https://mathiasbynens.be/notes/mysql-utf8mb4)

## Summary

Never use `utf8` in MySQL — always use `utf8mb4` instead. Updating your databases and code might take some time, but it’s definitely worth the effort. Why would you arbitrarily limit the set of symbols that can be used in your database? Why would you lose data every time a user enters an astral symbol as part of a comment or message or whatever it is you store in your database? There’s no reason not to strive for full Unicode support everywhere. Do the right thing, and use utf8mb4. 🍻
