---
layout: post
title: "How to disable Synology MariaDB password policy"
date: 2019-05-08
tags: ["MariaDB", "MySQL", "Synology"]
categories: [MariaDB]
description: "With release 10.3.21 Synology added an annoying password policy plugin that loads with Maria DB. And this shows how to disable Synology MariaDB password policy"
---

With release 10.3.21 Synology added an annoying password policy plugin that loads with Maria DB.
If you'd like to change your complex password back to something for example without upper case characters, do the following:

## Stop Maria DB service

Either via Package Center or via command line:

```
synoservice --stop pkgctl-MariaDB10
```

## Edit /var/packages/MariaDB10/target/usr/local/mariadb10/etc/mysql/my.cnf

Comment out (add # sign) for:

```ini
#synology_password_check = FORCE_PLUS_PERMANENT
#plugin_load_add = synology_password_check
```

- The first line tells mysql that the plugin named `synology_password_check` must not be uninstalled.
- The second line actually loads the plugin on sql startup.

## Start Maria DB service

Either via Package Center or via command line:

```
synoservice --start pkgctl-MariaDB10
```
