---
layout: post
title: "使用Let's Encrypt免费证书让你的网站SSL化"
date: 2021-05-09
tags: ["Ubuntu", "Ubuntu 20.04", "Nignx", "SSL", "Github Pages"]
categories: [Nginx]
description: "使用Let's Encrypt免费证书让你的网站SSL化，并且启用自动续期"
---

## 说明

[Let's Encrypt](https://letsencrypt.org/)是一个免费颁发SSL证书的非盈利组织，其颁布的证书已被Chrome等主流浏览器认可。GitHub Pages站点默认使用的SSL证书也是[Let's Encrypt](https://letsencrypt.org/)提供的。

此文主要解决：
1. 如何使用[Let's Encrypt](https://letsencrypt.org/)为我们的域名颁发证书
2. 颁发的证书如何自动更新有效时间

## About certbot

本文使用Certbot完成HTTPS的自动颁发和续期。其[官方介绍](https://certbot.eff.org/about/)如下：

> Certbot is a free, open source software tool for automatically using Let’s Encrypt certificates on manually-administrated websites to enable HTTPS.  
> Certbot is made by the Electronic Frontier Foundation (EFF), a 501(c)3 nonprofit based in San Francisco, CA, that defends digital privacy, free speech, and innovation.  

## 使用环境

- Web服务器：Nginx 1.18.0
- 宿主服务器：Ubuntu 20.04

说明：如果不是使用的以上两个服务器或版本，可使用Certbot官网选择对应服务器和版本。

## 安装过程

### SSH into the server
SSH into the server running your HTTP website as a user with sudo privileges.

### Install snapd
You'll need to install snapd and make sure you follow any instructions to enable classic snap support.
Follow these [instructions on snapcraft's site to install snapd](https://snapcraft.io/docs/installing-snapd).

> If you’re running Ubuntu 16.04 LTS (Xenial Xerus) or later, including Ubuntu 18.04 LTS (Bionic Beaver), Ubuntu 18.10 (Cosmic Cuttlefish) and Ubuntu 19.10 (Eoan Ermine), you don’t need to do anything. Snap is already installed and ready to go.

### Ensure that your version of snapd is up to date
Execute the following instructions on the command line on the machine to ensure that you have the latest version of snapd.

```shell
sudo snap install core; sudo snap refresh core
```

### Remove certbot-auto and any Certbot OS packages
If you have any Certbot packages installed using an OS package manager like apt, dnf, or yum, you should remove them before installing the Certbot snap to ensure that when you run the command certbot the snap is used rather than the installation from your OS package manager.

```shell
sudo apt-get remove certbot, sudo dnf remove certbot
```

If you previously used Certbot through the certbot-auto script, you should also remove its installation by following the instructions [here](https://certbot.eff.org/docs/uninstall.html).

### Install Certbot

Run this command on the command line on the machine to install Certbot.

```shell
sudo snap install --classic certbot
```

### Prepare the Certbot command

Execute the following instruction on the command line on the machine to ensure that the `certbot` command can be run.

```shell
sudo ln -s /snap/bin/certbot /usr/bin/certbot
```

### Get certificates and/or install certificates

#### Get and Install
Run this command to get a certificate and have Certbot edit your Nginx configuration automatically to serve it, turning on HTTPS access in a single step.

```shell
sudo certbot --nginx
```

#### Just get a certificate
If you're feeling more conservative and would like to make the changes to your Nginx configuration by hand, run this command.

```shell
sudo certbot certonly --nginx
```

### Test automatic renewal
The Certbot packages on your system come with a cron job or systemd timer that will renew your certificates automatically before they expire. You will not need to run Certbot again, unless you change your configuration. You can test automatic renewal for your certificates by running this command:

```shell
sudo certbot renew --dry-run
```

The command to renew certbot is installed in one of the following locations:
```
/etc/crontab/
/etc/cron.*/*
systemctl list-timers
```

My timers would be:

```
root@us:/var/log/nginx# systemctl list-timers
NEXT                        LEFT          LAST                        PASSED      UNIT                         ACTIVATES
Sun 2021-05-09 13:20:00 CST 1h 57min left Sun 2021-05-09 01:22:05 CST 10h ago     snap.certbot.renew.timer     snap.certbot.renew.service
...
...
```

### Confirm that Certbot worked
To confirm that your site is set up properly, visit https://yourwebsite.com/ in your browser and look for the lock icon in the URL bar.
