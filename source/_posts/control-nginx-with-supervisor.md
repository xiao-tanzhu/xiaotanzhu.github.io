---
layout: post
title: Controlling Nginx with Supervisor
date: 2021-07-17
tags: ["Nginx", "Supervisor", "Linux"]
categories: Linux
description: This example shows us how to control nginx with supervisor so when nginx goes down for whatever reason, supervisor will bring it up again as a "foreground" process not as a "daemon".
---

# Important

Supervisor uses `daemon off;` to start Nginx. The `daemon off;` directive tells Nginx to stay in the foreground. The `sudo service nginx stop/start/restart` commands won't work. For more information read [Stack Overflow](https://stackoverflow.com/questions/25970711/what-is-the-difference-between-nginx-daemon-on-off-option/34821579#34821579) answer.

# Current nginx process

As you can see below, Nginx is running in "daemon on" mode.

```
root@us:~# ps aux | grep nginx
root       12527  0.0  0.2 132576  1064 ?        Ss   09:52   0:00 nginx: master process /usr/sbin/nginx -g daemon on; master_process on;
www-data   12529  0.2  1.6 134176  8092 ?        S    09:52   0:07 nginx: worker process
www-data   12530  0.0  0.7 132808  3800 ?        S    09:52   0:00 nginx: cache manager process
root       13154  0.0  0.1   8900   660 pts/0    S+   10:38   0:00 grep --color=auto nginx
```

# Install & Setup supervisor

## Install

```bash
sudo apt update
sudo apt install supervisor
```

## Setup configuration for Nignx

Add `nginx.conf` file: `/etc/supervisor/conf.d/nginx.conf`

```
[program:nginx]
command=/usr/sbin/nginx -g "daemon off;"
autostart=true
autorestart=true
startretries=5
numprocs=1
startsecs=0
process_name=%(program_name)s_%(process_num)02d
stderr_logfile=/var/log/supervisor/%(program_name)s_stderr.log
stderr_logfile_maxbytes=10MB
stdout_logfile=/var/log/supervisor/%(program_name)s_stdout.log
stdout_logfile_maxbytes=10MB
```

Let supervisor know about new config.

```bash
$ sudo supervisorctl reread
nginx: available
```

Stop and disable the Nginx service

```bash
sudo systemctl stop nginx
sudo systemctl disable nginx
```

Let supervisor start nginx service.

```bash
$ sudo supervisorctl update
nginx: added process group
```

Verify if supervisor started nginx service.

```
$ sudo supervisorctl
nginx:nginx_00                   RUNNING   pid 18429, uptime 0:01:47
supervisor>
```

# Check

As you can see below, Nginx is running in "daemon off" mode.

```
root@us:~# ps aux | grep nginx
root         692  0.3  1.5 132576  7768 ?        S    10:56   0:00 nginx: master process /usr/sbin/nginx -g daemon off;
www-data     703  1.1  1.4 133140  6884 ?        S    10:56   0:00 nginx: worker process
www-data     704  0.0  0.7 132808  3620 ?        S    10:56   0:00 nginx: cache manager process
www-data     708  0.0  0.7 132808  3620 ?        S    10:56   0:00 nginx: cache loader process
root         800  0.0  0.1   8900   736 pts/0    S+   10:57   0:00 grep --color=auto nginx
```
