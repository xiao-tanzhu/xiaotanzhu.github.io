---
layout: post
title: Mongo常用命令
tags:
- Mongo
categories: 分布式系统
description: Mongo commands
---
###根据日期查找
```javascript
db.getCollection('webExceptionRecord').find({createdDate: {"$gte":ISODate("2016-03-31T12:48:25.040+08:00")}})
```

###根据日期倒序排序
```javascript
db.getCollection('webExceptionRecord').find({}).sort({createdDate:-1})
```

###查询“不等于”
```javascript
db.getCollection('webExceptionRecord').find({version: {$not:/v5\.0\.5/}})
```
`$not`后边必须跟正则表达式或文档（regex or document）

```javascript
db.getCollection('webExceptionRecord').find({version:{$nin:['v5.0.5']}})
```
`$nin`: not in

```javascript
db.getCollection('appStatisticsActiveUser').find({count:{$not: {$lt:100}}})
```

###查询“包含”
```javascript
db.getCollection('webExceptionRecord').find({version: {$regex:/.*5\.0\.5.*/}})
```

###查询“不包含”
```javascript
db.getCollection('webExceptionRecord').find({version: {$not:/.*v5\.0\.5.*/}})
```
<!-- more -->
