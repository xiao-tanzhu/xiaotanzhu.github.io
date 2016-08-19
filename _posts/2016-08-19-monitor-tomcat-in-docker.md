---
layout: post
title: 使用Moint监控Docker中运行的Web服务器
tags:
- 监控
- Moint
- Docker
categories: 运维
description: 使用Moint监控Docker中运行的Web服务器
---
Monit是一个Linux系统中常用的监控软件，可以监控CPU、内存、文件系统、TCP端口、UDP端口、HTTP等基本上所有的资源。可以参考其官方网站：https://mmonit.com/monit/。

本次要对Docker（Docker中运行了一个Web服务器）进行监控，需要满足以下几个需求：

1. 当Docker挂掉的时候自动启动；
2. 当Docker中的Web服务器300秒内无响应的时候重启Docker；
3. 当Docker发生异常或者重启时，发送邮件警告通知。

## 安装Moint软件

Moint的安装比较容易，如果使用Ubuntu/Debian，则可以直接执行：
```bash
sudo apt-get install monit
```
如果使用源代码编译安装，则是经典三部曲：
```bash
./configure
make
make install
```

## 基本配置

这里只提及几个比较重要的设置，其他设置可以参考Linux手册。

### 监控进程检查周期，默认为120秒检查一次
```ini
###############################################################################
## Global section
###############################################################################
##
## Start Monit in the background (run as a daemon):
#
  set daemon 30            # check services at 30-seconds intervals
#   with start delay 120   # optional: delay the first check by 4-minutes (by
#                          # default Monit check immediately after Monit start)
#
```
### 使用本地Sendmail发送邮件
```ini
## Set the list of mail servers for alert delivery. Multiple servers may be 
## specified using a comma separator. If the first mail server fails, Monit 
# will use the second mail server in the list and so on. By default Monit uses 
# port 25 - it is possible to override this with the PORT option.
#
# set mailserver mail.bar.baz,               # primary mailserver
#                backup.bar.baz port 10025,  # backup mailserver on port 10025
#                localhost                   # fallback relay
 set mailserver localhost               # primary mailserver
```
### 邮件发件人，我用来区别是哪台服务器发送的报警
```ini
## You can override this message format or parts of it, such as subject
## or sender using the MAIL-FORMAT statement. Macros such as $DATE, etc.
## are expanded at runtime. For example, to override the sender, use:
#
 set mail-format { from: node1@anying.me }
```
### 报警邮件接收人
```ini
## You can set alert recipients whom will receive alerts if/when a 
## service defined in this file has errors. Alerts may be restricted on 
## events by using a filter as in the second example below. 
#
 set alert fify@xiaotanzhu.com                       # receive all alerts
```
### 监控的Web服务
```ini
## Monit has an embedded web server which can be used to view status of 
## services monitored and manage services from a web interface. See the
## Monit Wiki if you want to enable SSL for the web server. 
#
 set httpd port 2812 and
    use address localhost  # only accept connection from localhost
    allow localhost        # allow localhost to connect to the server and
    allow admin:monit      # require user 'admin' with password 'monit'
    allow @monit           # allow users of group 'monit' to connect (rw)
    allow @users readonly  # allow users of group 'users' to connect readonly
```

## 监听Docker服务

在`/etc/monit/conf.d`目录下增加监控文件`irenshi-web`：
```
check process irenshi-web matching "docker-proxy.*-host-port 8081.*"
  start program "/usr/bin/docker start irenshi-web"
  stop  program "/usr/bin/docker stop irenshi-web"
  if failed host localhost port 8081 protocol http request "/web/" for 10 times within 12 cycles then restart
```

其中：

- `irenshi-web`为监控项目名称，这个名称将显示在报警邮件中。另外，启动和关闭项目也需要使用该名称，见[需要停止Docker时](#需要停止Docker时)
- `matching "docker-proxy.*-host-port 8081.*"`通过正则表达式来识别进程。通常情况下可以使用pidfile来识别进程，但Docker的实例没有进程pidfile，所以我们通过正则表达式匹配进程。Monit判断进程是否存在的时候会根据该正则表达式进行进程名搜索。
- `start program`启动进程的方法，这里必须使用完整路径`/usr/bin/docker`，否则会无法找到命令
- `stop program`关闭进程的方法。当Monit执行重启的时候，会先调用`stop program`，然后再调用`start program`
- `if failed host localhost port 8081 protocol http request "/web/" for 10 times within 12 cycles then restart`
	1. `if failed`指定访问失败时的情形。当进程还在，但是Web服务器已经挂掉时，会触发`fail`
	2. `localhost port 8081`监听本地的8081端口，这是Docker映射给主机的端口
	3. `protocol http request "/web/"`使用HTTP协议请求`/web/`这个页面
	4. `for 10 times within 12 cycles`出现次数设定：在12个检测周期中，如果出现了10次无法访问
	5. `then restart`满足以上条件时执行的操作，这里执行`restart`

备注：Docker的完整进程名为
```
docker-proxy -proto tcp -host-ip 0.0.0.0 -host-port 8081 -container-ip 192.168.42.253 -container-port 8080
```
由于同一台机器上启动了若干个Docker，为防止干扰我们采用了取巧的方法，用`-host-port 8086`来识别进程实例。

## 需要停止Docker时

如果使用`docker stop irenshi-web`命令停止Docker的话，Monit检测到应用不存在会立即重新启动。所以应该使用以下命令关闭Docker：

```bash
monit -c /etc/monit/monitrc stop irenshi-web
```

如果需要重新启动，则对应执行以下命令：
```bash
monit -c /etc/monit/monitrc start irenshi-web
```
