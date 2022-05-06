---
layout: post
title: Ubuntu安装php+nginx环境
date: 2022-05-07
tags:
- Ubuntu
- php
- Nginx
categories: Ubuntu
description: 在Ubuntu 20.04中安装Nginx + php7.4，并安装基础扩展。
---

## 环境

- Ubuntu: 20.04
- Nginx: 1.18.0
- PHP: 7.4

## 安装Nginx

```shell
sudo apt install nginx -y
```

## 安装PHP

Ubuntu 20.04官方源自带的是PHP7.4，直接安装这个版本。

### 安装php
```shell
sudo apt install php-fpm
```

### 启动php
```shell
/etc/init.d/php7.4-fpm start
```

设置自动启动
```shell
sudo systemctl enable php7.4-fpm
```

## 启用PHP

在nginx的server域增加以下代码（或者直接在default站点中取消注释掉以下代码）：
```conf
# pass PHP scripts to FastCGI server
#
location ~ \.php$ {
        include snippets/fastcgi-php.conf;

        # With php-fpm (or other unix sockets):
        fastcgi_pass unix:/var/run/php/php7.4-fpm.sock;
        # With php-cgi (or other tcp sockets):
        #fastcgi_pass 127.0.0.1:9000;
}
```

## 安装扩展

### 安装：mysqli_connect

```
sudo apt install php-mysql
```

修改`/etc/php/7.4/fpm/php.ini`，取消以下行的注释：
```ini
extension=mysqli
```

重启php：
```
sudo /etc/init.d/php7.4-fpm restart
```

### 安装：mb_strlen

```
sudo apt install php-mbstring
```

修改`/etc/php/7.4/fpm/php.ini`，取消以下行的注释：
```ini
extension=mbstring
```

重启php：
```
sudo /etc/init.d/php7.4-fpm restart
```

### 安装：bcmath（含bccomp）

```
sudo apt install php-bcmath
```

重启php：
```
sudo /etc/init.d/php7.4-fpm restart
```

### 安装：curl

```
sudo apt install php-curl
```

修改`/etc/php/7.4/fpm/php.ini`，取消以下行的注释：
```ini
extension=curl
```

重启php：
```
sudo /etc/init.d/php7.4-fpm restart
```

### 安装：zip

```
sudo apt install php-zip
```

重启php：
```
sudo /etc/init.d/php7.4-fpm restart
```

### 安装：gd

```
sudo apt install php-gd
```

修改`/etc/php/7.4/fpm/php.ini`，取消以下行的注释：
```ini
extension=gd2
```

重启php：
```
sudo /etc/init.d/php7.4-fpm restart
```

### 安装：dom

```
sudo apt install php-xml
```

重启php：
```
sudo /etc/init.d/php7.4-fpm restart
```
