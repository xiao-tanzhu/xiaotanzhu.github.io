---
layout: post
title: Install the latest Node.js on Ubuntu 14.04
tags:
- Nodejs
- Ubuntu
categories: Ubuntu
description: Install the latest node.js on Ubuntu
---

> **NOTE:** If you are using Ubuntu Precise or Debian Wheezy, you might want to read about [running Node.js >= 4.x on older distros](https://github.com/nodesource/distributions/blob/master/OLDER_DISTROS.md "running Node.js >= 4.x on older distros").


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
