---
layout: post
title: Mongo常用命令
tags:
- Mongo
categories: 分布式系统
description: Mongo commands
---
###根据日期查找
```
db.getCollection('webExceptionRecord').find({createdDate: {"$gte":ISODate("2016-03-31T12:48:25.040+08:00")}})
```

###根据日期倒序排序
```
db.getCollection('webExceptionRecord').find({}).sort({createdDate:-1})
```
<!-- more -->
