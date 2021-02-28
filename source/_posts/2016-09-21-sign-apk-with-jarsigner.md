---
layout: post
title: Android包（apk）手动签名
date: 2016-09-21
tags:
- apk
- 安卓
- 签名
categories: 安卓
description: Android包（apk）手动签名
---

语法：
```bash
jarsigner -verbose -keystore <keystore file> -signedjar <output signed file> <apk to be signed> <alias>
```

举例：
```bash
jarsigner -verbose -keystore IrenshiRelease.keystore -signedjar signed.apk unsign.apk i人事
```
