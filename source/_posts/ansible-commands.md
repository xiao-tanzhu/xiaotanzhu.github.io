---
layout: post
title: Ansible常用命令
date: 2020-05-17
tags:
- Ansible
categories: 运维
description: Ansible在批量服务器管理中常用的命令
---

## Host匹配方式
```ini
web_server:!http1:&http2
http*:web_server
web_server[0]
web_server[0:3]
'~(web|http)*'
```

## 用户名密码验证方式
```bash
ansible web_server -a "shell命令" -k
ansible web_server -a "shell命令" -u username -k
ansible web_server -a "shell命令" -u username --sudo [--ask-sudo-pass]
```

## 多进程并发执行
```bash
ansible web_server -a "/sbin/reboot" -f 10
```

## Ansible模块
Shell操作
```bash
ansible web_server -m shell -a "grep '/sbin/nologin' /etc/passwd | wc -l"
ansible web_server -m shell -a 'echo $PATH'
```

文件操作
```bash
ansible web_server -m copy -a "src=/etc/hosts dest=/tmp/"
ansible web_server -m file -a "dest=/tmp/hosts mode=600 owner=xuad group=xuad" # chmod chown
ansible web_server -m file -a "dest=/tmp/test mode=755 owner=xuad group=xuad state=directory" # mkdir -p /tmp/test
ansible web_server -m file -a "dest=/tmp/test.txt mode=755 owner=xuad group=xuad state=touch" # touch /tmp/test.txt
ansible web_server -m file -a "src=/root/test.txt dest=/tmp/test.txt state=link" # ln -s
ansible web_server -m file -a "dest=/tmp/test.txt state=absent" # rm -rf
```

软件安装
```bash
ansible web_server -m yum -a "name=wget state=present" # check if exist
ansible web_server -m yum -a "name=wget state=latest" # upgrade
ansible web_server -m yum -a "name=wget state=removed" # remove
ansible web_server -m yum -a "name=wget state=absent" # check if nonexist
ansible web_server -m yum -a "name=wget state=installed" # install
```

用户名密码
```bash
ansible web_server -m user -a "name=andy state=absent" # remove user
ansible web_server -m user -a "name=andy state=present" # create on absence
ansible web_server -m user -a 'name=andy password="&lt;crypted password here&gt;"'
```
