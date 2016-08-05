---
layout: post
title: Nginx加速：启用gzip压缩和浏览器缓存
tags:
- Nginx
- 转载
categories: Web
description: Nginx是一个高性能的Web服务器，之前也写过一些关于nginx的文章。为了提高博客的响应速度，可以从设置nginx的gzip和浏览器缓存这两个方面入手。为字体开启gzip和缓存能大大减少带宽的消耗。
---
Nginx是一个高性能的Web服务器，之前也写过一些关于nginx的文章。为了提高博客的响应速度，可以从设置nginx的gzip和浏览器缓存这两个方面入手。为字体开启gzip和缓存能大大减少带宽的消耗。

### 开启gzip

#### Nginx配置参数

```
# 开启gzip
gzip on;

# 启用gzip压缩的最小文件，小于设置值的文件将不会压缩
gzip_min_length 1k;

# gzip 压缩级别，1-10，数字越大压缩的越好，也越占用CPU时间，后面会有详细说明
gzip_comp_level 2;

# 进行压缩的文件类型。javascript有多种形式。其中的值可以在 mime.types 文件中找到。
gzip_types text/plain application/javascript application/x-javascript text/css application/xml text/javascript application/x-httpd-php image/jpeg image/gif image/png;

# 是否在http header中添加Vary: Accept-Encoding，建议开启
gzip_vary on;

# 禁用IE 6 gzip
gzip_disable "MSIE [1-6]\.";
```

#### `gzip_comp_level`参数的推荐值

来自[serverfault][http://serverfault.com/questions/253074/what-is-the-best-nginx-compression-gzip-level]的一个讨论，有人做了压缩比的实验，结果如下：

**压缩HTML文件**
>级别	压缩之后的大小（压缩比）
>0    55.38 KiB (100.00% of original size)
>1    11.22 KiB ( 20.26% of original size)
>2    10.89 KiB ( 19.66% of original size)
>3    10.60 KiB ( 19.14% of original size)
>4    10.17 KiB ( 18.36% of original size)
>5     9.79 KiB ( 17.68% of original size)
>6     9.62 KiB ( 17.37% of original size)
>7     9.50 KiB ( 17.15% of original size)
>8     9.45 KiB ( 17.06% of original size)
>9     9.44 KiB ( 17.05% of original size)

**压缩JS文件（jQuery 1.8.3）
>级别	压缩之后的大小（压缩比）
>0    261.46 KiB (100.00% of original size)
>1     95.01 KiB ( 36.34% of original size)
>2     90.60 KiB ( 34.65% of original size)
>3     87.16 KiB ( 33.36% of original size)
>4     81.89 KiB ( 31.32% of original size)
>5     79.33 KiB ( 30.34% of original size)
>6     78.04 KiB ( 29.85% of original size)
>7     77.85 KiB ( 29.78% of original size)
>8     77.74 KiB ( 29.73% of original size)
>9     77.75 KiB ( 29.74% of original size)

从图中可以看出`gzip_comp_level`大于2时效果并不是很明显。所以可以将值设置为1或者2。

#### 选择性的开启字体压缩
为静态资源开启缓存能够较少服务器带宽的消耗，特别是在css中使用字体时，同时配合gzip压缩能够大大减少下载字体造成的带宽影响。

**需要注意**：只需要为ttf、otf和svg字体启用gzip，对其他字体格式进行gzip压缩时效果不明显。
```
gzip_types  font/ttf font/otf image/svg+xml
```
各种字体类型压缩效果可以参考引用原文[reference](http://www.darrenfang.com/2015/01/setting-up-http-cache-and-gzip-with-nginx/ "引用原文")。

**字体压缩总结**
|扩展名|是否压缩|Content-type|
|.eot|否|application/vnd.ms-fontobject|
|.ttf|是|font/ttf|
|.otf|是|font/opentype|
|.woff|否|font/x-woff|
|.svg|是|image/svg+xml|

### 开启浏览器缓存

#### Nginx配置

```
location ~* ^.+\.(ico|gif|jpg|jpeg|png)$ { 
        access_log   off; 
        expires      30d;
}

location ~* ^.+\.(css|js|txt|xml|swf|wav)$ {
    access_log   off;
    expires      24h;
}

location ~* ^.+\.(html|htm)$ {
        expires      1h;
}
```
其中的缓存时间可以自己根据需要修改。

#### 开启字体缓存
```
location ~* ^.+\.(eot|ttf|otf|woff|svg)$ {
        access_log   off;
        expires max;
}
```



