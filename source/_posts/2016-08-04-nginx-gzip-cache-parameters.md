---
layout: post
title: Nginx的浏览器缓存和Gzip压缩参数
date: 2016-08-04
tags:
- Nginx
- Browser Caching
- Gzip
categories: Web服务器
description: Nginx的浏览器缓存和Gzip压缩参数
---
### 浏览器缓存（Browser Caching）
浏览器缓存是为了加速浏览并节约网络资源，浏览器在用户磁盘上对最近请求过的文档进行存储。nginx可以通过 expires 指令来设置浏览器的Header。

>语法： `expires [time|epoch|max|off]`
>默认值： `expires off`
>作用域： `http`, `server`, `location`

使用本指令可以控制HTTP应答中的“Expires”和“Cache-Control”的头标，（起到控制页面缓存的作用）。可以在time值中使用正数或负数。“Expires”头标的值将通过当前系统时间加上您设定的`time`值来获得。
- `epoch`指定“Expires”的值为 1 January, 1970, 00:00:01 GMT。
- `max`指定“Expires”的值为 31 December 2037 23:59:59 GMT，“Cache-Control”的值为10年。
- `-1`指定“Expires”的值为 服务器当前时间 -1s,即永远过期

例子：图片缓存30天
```nginx
# 图片缓存30天
location ~.*\.(jpg|png|jpeg)$ {
   expires 30d;
}
# js css缓存一小时
location ~.*\.(js|css)?$ {
   expires 1h;
}
```

### Nginx gzip压缩
使用 gzip 压缩可以降低网站带宽消耗，同时提升访问速度。主要在nginx服务端将页面进行压缩，然后在浏览器端进行解压和解析，目前大多数流行的浏览器都迟滞gzip格式的压缩，所以不用担心。默认情况下，Nginx的gzip压缩是关闭的，同时，Nginx默认只对text/html进行压缩。

主要配置如下：
```nginx
gzip  on;#开启
gzip_http_version 1.0;#默认1.1
gzip_vary on;
gzip_comp_level 6;
gzip_proxied any;
gzip_types text/plain text/html text/css application/json application/x-javascript text/xml application/xml application/xml+rss text/javascript;#压缩的文件类型

gzip_buffers 16 8k;#设置gzip申请内存的大小,其作用是按块大小的倍数申请内存空间设置gzip申请内存的大小,其作用是按块大小的倍数申请内存空间

# Disable gzip for certain browsers.
gzip_disable “MSIE [1-6].(?!.*SV1)”;#ie6不支持gzip，需要禁用掉ie6，可恶啊!!!!
```
**注意：** 其中的`gzip_http_version`的设置，它的默认值是`1.1`，就是说对HTTP/1.1协议的请求才会进行gzip压缩。如果我们使用了`proxy_pass`进行反向代理，那么nginx和后端的`upstream server`之间是用HTTP/1.0协议通信的。

**参数说明**

`gzip`决定是否开启gzip模块

>param:on|off

`gzip_buffers`设置gzip申请内存的大小,其作用是按块大小的倍数申请内存空间

>param1:int 增加的倍数
>param2:int(k) 后面单位是k

`gzip_comp_level`设置gzip压缩等级，等级越底压缩速度越快文件压缩比越小，反之速度越慢文件压缩比越大

>param:1-9

`gzip_min_length`当返回内容大于此值时才会使用gzip进行压缩,以K为单位,当值为0时，所有页面都进行压缩

>param:int

`gzip_http_version`用于识别http协议的版本，早期的浏览器不支持gzip压缩，用户会看到乱码，所以为了支持前期版本加了此选项,目前此项基本可以忽略

>param: 1.0|1.1

`gzip_proxied`Nginx做为反向代理的时候启用，

>param:off|expired|no-cache|no-sotre|private|no_last_modified|no_etag|auth|any]
- off – 关闭所有的代理结果数据压缩
- expired – 启用压缩，如果header中包含”Expires”头信息
- no-cache – 启用压缩，如果header中包含”Cache-Control:no-cache”头信息
- no-store – 启用压缩，如果header中包含”Cache-Control:no-store”头信息
- private – 启用压缩，如果header中包含”Cache-Control:private”头信息
- no_last_modified – 启用压缩，如果header中包含”Last_Modified”头信息
- no_etag – 启用压缩，如果header中包含“ETag”头信息
- auth – 启用压缩，如果header中包含“Authorization”头信息
- any – 无条件压缩所有结果数据

`zip_types`设置需要压缩的MIME类型,非设置值不进行压缩

>param:text/html|application/x-javascript|text/css|application/xml

`gzip_vary on;`和http头有关系，加个vary头，给代理服务器用的，有的浏览器支持压缩，有的不支持，所以避免浪费不支持的也压缩，所以根据客户端的HTTP头来判断，是否需要压缩。