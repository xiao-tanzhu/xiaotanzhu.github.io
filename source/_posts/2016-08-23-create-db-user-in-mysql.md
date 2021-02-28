---
layout: post
title: 在MySQL中创建库和用户
date: 2016-08-23
tags:
- MySQL
categories: 数据库
description: 在MySQL中创建库和用户
---

### 创建数据库
```sql
CREATE DATABASE `irenshi` DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
```

### 创建用户
```sql
CREATE USER 'irenshi'@'%' IDENTIFIED BY 'irenshi';
```

### 授权数据库访问
```sql
GRANT ALL PRIVILEGES ON irenshi.* TO 'irenshi'@'%';
```
