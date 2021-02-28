---
layout: post
title: 使用Goaccess监控Nginx的运行状态
date: 2016-08-12
tags:
- Nginx
- 监控
- Goaccess
categories: 运维
description: 使用Goaccess监控Nginx的运行状态
---
## 最终启动命令

如果你只需要知道最终的启动命令，可以跳过下边所有内容。启动命令：
```bash
goaccess -f /var/log/nginx/access.log -o /data/www/access/index.html --real-time-html --ws-url=access1.xiaotanzhu.com
```

## 环境准备
系统版本：
```
root@www:~# lsb_release -a
No LSB modules are available.
Distributor ID: Ubuntu
Description:    Ubuntu 14.04.2 LTS
Release:        14.04
Codename:       trusty
```
Nginx日志配置
```nginx
log_format access '$remote_addr - $remote_user [$time_local] "$request" $status $body_bytes_sent "$http_referer" "$http_user_agent" $http_x_forwarded_for $request_time $upstream_response_time';
```
Nginx日志样例
```
180.98.186.40 - - [12/Aug/2016:14:39:43 +0800] "POST /api/heartbeat/alive HTTP/1.1" 200 73 "-" "okhttp/3.3.1" - 0.002 0.002
202.112.25.39 - - [12/Aug/2016:14:39:43 +0800] "POST /api/attendance/signin/time/detail/v2 HTTP/1.1" 200 110 "-" "okhttp/3.3.1" - 0.197 0.197
180.98.186.40 - - [12/Aug/2016:14:39:43 +0800] "POST /api/message/unread/v5 HTTP/1.1" 200 2322 "-" "okhttp/3.3.1" - 0.085 0.085
180.98.186.40 - - [12/Aug/2016:14:39:43 +0800] "POST /api/common/app/advertisement/v2 HTTP/1.1" 200 59 "-" "okhttp/3.3.1" - 0.076 0.076
180.98.186.40 - - [12/Aug/2016:14:39:43 +0800] "POST /api/common/version/v1 HTTP/1.1" 200 532 "-" "okhttp/3.3.1" - 0.097 0.097
180.98.186.40 - - [12/Aug/2016:14:39:43 +0800] "POST /api/honour/queryFavoriteImageList/v1 HTTP/1.1" 200 1172 "-" "okhttp/3.3.1" - 0.108 0.108
180.98.186.40 - - [12/Aug/2016:14:39:43 +0800] "POST /api/log/info HTTP/1.1" 200 47 "-" "okhttp/3.3.1" - 0.119 0.119
202.112.25.39 - - [12/Aug/2016:14:39:44 +0800] "POST /api/attendance/signin/time/detail/v2 HTTP/1.1" 200 110 "-" "okhttp/3.3.1" - 0.235 0.235
202.112.25.39 - - [12/Aug/2016:14:39:44 +0800] "POST /api/attendance/signin/condition/v4 HTTP/1.1" 200 305 "-" "okhttp/3.3.1" - 0.134 0.134
202.112.25.39 - - [12/Aug/2016:14:39:44 +0800] "POST /api/attendance/signin/time/detail/v2 HTTP/1.1" 200 110 "-" "okhttp/3.3.1" - 0.165 0.165
```
## 安装

安装方法官方网站给的很清楚，传送门：https://goaccess.io/download

这里节选出Ubuntu上的安装方法
```bash
echo "deb http://deb.goaccess.io/ $(lsb_release -cs) main" | sudo tee -a /etc/apt/sources.list.d/goaccess.list
wget -O - https://deb.goaccess.io/gnugpg.key | sudo apt-key add - 
sudo apt-get update
sudo apt-get install goaccess
```
最简单的使用方法：
```bash
goaccess -f /var/log/nginx/access.log
```
分析完成之后可以看到一个终端显示（libncurses）的页面，也可以对这个页面进行交互，具体的交互方式可以参考`man goaccess`。

## 配置

如果使用Nginx默认的日志格式，那么Goaccess不需要任何设置即可使用。但是我们的日志为自定义格式，在默认日志的最后增加了响应时间`$request_time`和upstream的响应时间`$upstream_response_time`，如下：
```nginx
log_format access '$remote_addr - $remote_user [$time_local] "$request" $status $body_bytes_sent "$http_referer" "$http_user_agent" $http_x_forwarded_for $request_time $upstream_response_time';
```

于是需要修改Goaccess监控的日志的格式，原格式
```diff
- log-format %h %^[%d:%t %^] "%r" %s %b "%R" "%u"
+ log-format %h %^[%d:%t %^] "%r" %s %b "%R" "%u" %^ %^ %T
```
最后三个元素`%^ %^ %T`，`%^`表示忽略，`%T`表示“以秒为单位的响应时间，精确到毫秒”。

完整的日志格式可以参考`man goaccess`，这里节选其中的日志格式部分：

       log-format
              The log-format variable followed by a space or \t , specifies the log format string.

       %x     A date and time field matching the time-format and date-format variables. This is used when a timestamp is given instead of the date and time being in two separated vari‐
              ables.
       %t     time field matching the time-format variable.
       %d     date field matching the date-format variable.
       %v     The canonical Server Name of the server serving the request (Virtual Host).
       %h     host (the client IP address, either IPv4 or IPv6)
       %r     The request line from the client. This requires specific delimiters around the request (as single quotes, double quotes, or anything else) to be parsable. If not, we have
              to use a combination of special format specifiers as %m %U %H.
       %q     The query string.
       %m     The request method.
       %U     The URL path requested.
              Note:  If the query string is in %U, there is no need to use %q.  However, if the URL path, does not include any query string, you may use %q and the query string will be
              appended to the request.
       %H     The request protocol.
       %s     The status code that the server sends back to the client.
       %b     The size of the object returned to the client.
       %R     The "Referrer" HTTP request header.
       %u     The user-agent HTTP request header.
       %D     The time taken to serve the request, in microseconds as a decimal number.
       %T     The time taken to serve the request, in seconds with milliseconds resolution.
       %L     The time taken to serve the request, in milliseconds as a decimal number.

              Note: If multiple time served specifiers are used at the same time, the first option specified in the format string will take priority over the other specifiers.

       %^     Ignore this field.
       %~     Move forward through the log string until a non-space (!isspace) char is found.

       GoAccess requires the following fields:
              %h a valid IPv4/6
              %d a valid date
              %r the request


## 监控

实际监控还是用HTML来的方便，而且Goaccess号称实时监控，那这个特性肯定也要用一下了。

实时监控启动命令：
```bash
goaccess -f /var/log/nginx/access.log -o /data/www/access/index.html --real-time-html --ws-url=access1.xiaotanzhu.com
```

下边每个参数分别介绍一下。

1. `-f /var/log/nginx/access.log` 指定要分析的Nginx日志文件，没什么好说的
2. `-o /data/www/access/index.html` 指定输出的HTML文件
3. `--real-time-html` 启动实时HTML。这里需要注意的是，所谓实时并不是goaccess不停的更新HTML文件。事实上，HTML文件生成之后，就不再变化，而页面上是通过WebSocket对页面上的数据进行更新
4. `--ws-url=access1.irenshi.cn` 指定WebSocket的地址。由于我使用`access1.xiaotanzhu.com`访问生成的HTML，所以`--ws-url`必须指定为`access1.xiaotanzhu.com`，这样生成的HTML才知道应该去`ws://access1.xiaotanzhu.com:7890`访问这个WebSocket。
5. `--port=7890` 我们这里并没有使用这个参数。这个参数用来控制goaccess生成的WebSocket的端口。默认值：7890。

## 效果

### 监控界面

![](/upload/images/10.png)

我们可以看到其中的数据在实时变化。

### 资源占用

使用`top`命令查看内存占用情况如下：
```
  PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND
26727 root      20   0  219624  51136   1612 S   1.3  0.6   0:52.13 goaccess
```
现在来看处理百兆的日志时，资源还是比较省的。
