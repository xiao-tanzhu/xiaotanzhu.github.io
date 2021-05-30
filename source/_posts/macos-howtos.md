---
layout: post
title: "MacOS常见操作"
date: 2021-05-18
tags: ["MacOS", "MacBook"]
categories: [MacOS]
description: "目录：MacOS常见的一些配置、调整或软件"
---

## 常见配置

### 屏幕截图的默认存储位置调整

在mac终端执行命令：
```
defaults write com.apple.screencapture location ~/Desktop/idolaoxuPic
```

执行完后，需执行如下命令生效（当然，重启也是可以的）
```
killall SystemUIServer
```

> MacOS截屏快捷键
> 1. Command + Shift + 3：拍摄截屏
> 2. Command + Shift + 4：捕捉屏幕上的某一部分
> 3. Command + Shift + 5
>
> Reference：https://support.apple.com/zh-cn/HT201361
