---
layout: post
title: Java中的DelayQueue
date: 2020-06-28
tags: ["Java", "DelayQueue"]
categories: Java
description: Java中的DelayQueue
---

**面试官：好久不见啊，上次我们聊完了`PriorityBlockingQueue`，今天我们再来聊聊和它相关的`DelayQueue`吧？**

Hydra：就知道你前面肯定给我挖了坑，DelayQueue也是一个无界阻塞队列，但是和之前我们聊的其他队列不同，不是所有类型的元素都能够放进去，只有实现了Delayed接口的对象才能放进队列。Delayed对象具有一个过期时间，只有在到达这个到期时间后才能从队列中取出。


---

> 原文链接：https://mp.weixin.qq.com/s/XOToC0CUI4H6AFTSwKZMRw
