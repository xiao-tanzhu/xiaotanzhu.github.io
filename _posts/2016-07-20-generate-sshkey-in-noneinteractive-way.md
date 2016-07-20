---
layout: post
title: Generate SSH key in none-interactive way
tags:
- Shell
- Bash
categories: Linux
description: Generate SSH key in none-interactive way
---
We use ssh-keygen to generate ssh keys. Sometime we need to generate keys without any interaction during runtime. We have the following ways to do this.

##Use the full commands
Provide all parameters needed when running ssh-keygen, as
```bash
ssh-keygen -t rsa -N "" -f my.key
```
in which

- *`-N ""`* tells it to use an empty passphrase (the same as two of the enters in an interactive script)
- *`-f my.key`* tells it to store the key into my.key (change as you see fit).

##Provide "ENTER"s before running ssh-keygen
```bash
echo -e "\n\n\n" | ssh-keygen -t rsa
```
By using pipe, we actually enters 3 "ENTER"s while executing ssh-keygen, which uses the default values.

