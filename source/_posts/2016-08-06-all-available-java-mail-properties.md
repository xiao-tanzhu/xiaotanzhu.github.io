---
layout: post
title: Where to find all available Java mail properties?
date: 2016-08-06
tags:
- Java Mail
categories: Java
description: Where to find all available Java mail properties?
---

In the api is a reference to the properties for the specific sun protocol providers.

- `IMAP` https://javamail.java.net/nonav/docs/api/com/sun/mail/imap/package-summary.html
- `POP3` https://javamail.java.net/nonav/docs/api/com/sun/mail/pop3/package-summary.html
- `SMTP` https://javamail.java.net/nonav/docs/api/com/sun/mail/smtp/package-summary.html

These are also set on the session object but you use them on your own risk since in other mail implementations they are maybe not supported or they change in the future.