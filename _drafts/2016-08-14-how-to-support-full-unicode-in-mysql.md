---
layout: post
title: How to support full Unicode in MySQL databases
tags:
- MySQL
- utf8mb4
- Emoji
categories: 数据库
description: How to support full Unicode in MySQL databases
---

如果你完全从头建立一个数据库，那么事情简单了很多，直接参考：[创建支持emoji表情的MySQL数据库（utf8mb4）](/2016/08/14/create-mysql-database-with-utf8mb4.html)。

如果你的数据库已经在投入使用，那么可以参考文章的以下部分。

> **Reference**: https://mathiasbynens.be/notes/mysql-utf8mb4

## UTF-8

[The UTF-8 encoding](https://encoding.spec.whatwg.org/#utf-8) can represent every symbol in the Unicode character set, which ranges from `U+000000` to `U+10FFFF`. That’s 1,114,112 possible symbols. (Not all of these Unicode code points have been assigned characters yet, but that doesn’t stop UTF-8 from being able to encode them.)

UTF-8 is a variable-width encoding; it encodes each symbol using **one to four 8-bit bytes**. Symbols with lower numerical code point values are encoded using fewer bytes. This way, UTF-8 is optimized for the common case where ASCII characters and other [BMP symbols](https://mathiasbynens.be/notes/javascript-encoding#bmp) (whose code points range from `U+000000` to `U+00FFFF`) are used — while still allowing astral symbols (whose code points range from `U+010000` to `U+10FFFF`) to be stored.

## MySQL’s `utf8`

