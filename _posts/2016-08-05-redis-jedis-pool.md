---
layout: post
title: Redis提示Could not get a resource from the pool（jedis连接池配置）
tags:
- Redis
- Jedis
- 转载
categories: 分布式系统
description: Redis提示Could not get a resource from the pool（jedis连接池配置）
---

起初在JedisPool中配置了50个活动连接，但是程序还是经常报错：Could not get a resource from the pool

连接池刚开始是这样配置的：


#### Node.js v6.x:
```
# Using Ubuntu
curl -sL https://deb.nodesource.com/setup_6.x | sudo -E bash -
sudo apt-get install -y nodejs

# Using Debian, as root
curl -sL https://deb.nodesource.com/setup_6.x | bash -
apt-get install -y nodejs
```

#### Node.js v5.x:
```
# Using Ubuntu
curl -sL https://deb.nodesource.com/setup_5.x | sudo -E bash -
sudo apt-get install -y nodejs

# Using Debian, as root
curl -sL https://deb.nodesource.com/setup_5.x | bash -
apt-get install -y nodejs
```

#### Node.js v4.x:
```
# Using Ubuntu
curl -sL https://deb.nodesource.com/setup_4.x | sudo -E bash -
sudo apt-get install -y nodejs

# Using Debian, as root
curl -sL https://deb.nodesource.com/setup_4.x | bash -
apt-get install -y nodejs
```

For installations on other platforms, you can refer to: [https://github.com/nodesource/distributions#debinstall](https://github.com/nodesource/distributions#debinstall)
