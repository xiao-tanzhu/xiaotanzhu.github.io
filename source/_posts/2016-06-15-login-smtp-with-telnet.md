---
layout: post
title: 利用telnet进行SMTP的验证
tags:
- SMTP
- Telnet
- Email
date: 2016-06-15 12:00:00
updated: 2016-06-15 12:00:00
categories: 其他
description: 利用telnet进行SMTP的验证
---
先计算BASE64编码的用户名密码，认证登录需要用到：

>**fify@fify-Vostro-3902:~$ perl -MMIME::Base64 -e 'print encode_base64("wangjingfei");'**
d2FuZ2ppbmdmZWk=
**fify@fify-Vostro-3902:~$ perl -MMIME::Base64 -e 'print encode_base64("fakepassword");'**
ZmFrZXBhc3N3b3Jk

开始登录并使用SMTP发送邮件：

>**fify@fify-Vostro-3902:~$ telnet smtp.163.com 25**
>Trying 123.125.50.133...
>Connected to smtp.163.com.
>Escape character is '^]'.
>220 163.com Anti-spam GT for Coremail System (163com[20141201])
>**EHLO smtp.163.com** # 开始连接前握手
>250-mail
>250-PIPELINING
>250-AUTH LOGIN PLAIN
>250-AUTH=LOGIN PLAIN
>250-coremail 1Uxr2xKj7kG0xkI17xGrU7I0s8FY2U3Uj8Cz28x1UUUUU7Ic2I0Y2UFCvVepUCa0xDrUUUUj
>250-STARTTLS
>250 8BITMIME
>**AUTH LOGIN** # 开始登录，使用LOGIN方式
>334 dXNlcm5hbWU6
>**d2FuZ2ppbmdmZWk=** # 用户名的Base64
>334 UGFzc3dvcmQ6
>**ZmFrZXBhc3N3b3Jk** # 密码的Base64
>235 Authentication successful
>**MAIL FROM: &lt;wangjingfei@163.com&gt;** # 发件箱地址
>250 Mail OK
>**RCPT TO: &lt;wangjingfei@263.com&gt;** # 收件箱地址
>250 Mail OK
>**DATA** # 开始编写邮件正文
>354 End data with &lt;CR&gt;&lt;LF&gt;.&lt;CR&gt;&lt;LF&gt;
>**TO: obama@whitehouse.com** # 这里的数据可以随便写
>**FROM: xijinping@zhongnanhai.com** # 这里也是
>**SUBJECT: A very important conference.**
>
>**Please note that!**
>
>**.**
>
>250 Mail OK queued as smtp10,wKjADQ2ApxRnnqBE0CWaEw==.38326S3 # 返回250 表示发送成功。
>**NOOP** # 空语句，不执行任何操作，一般用来保持和服务器连接，不要掉线
>250 OK
>**QUIT** # 退出
>221 Closing connection. Good bye.
>Connection closed by foreign host.

