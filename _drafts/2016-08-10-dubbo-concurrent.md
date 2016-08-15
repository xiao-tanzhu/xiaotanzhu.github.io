---
layout: post
title: Dubbo服务器端并发数量
tags:
- Dubbo
- SOA
categories: 架构
description: Partial selection of Spring Data JPA
---

## 目标
1. 服务器端可以控制并发数量；
2. 客户端访问时，如果并发数量已经超出服务器控制数量，则多余的请求被阻塞，直到并发数量下降。

## 尝试

**测试流程：** 启动一个Dubbo服务器端，启动两个Dubbo客户端，然后同时使用两个客户端，各启动100个线程发送请求给服务器端。

### `executes`参数
设置Dubbo服务器端如下，服务器端并发数量为3：
```java
@Service(version = "1.2", executes = 3)
```
启动Dubbo服务器端，然后启动其中一个Dubbo客户端发送请求：只有前三个请求成功发送，之后的全部报错：
```
Exception in thread "pool-2-thread-97" com.alibaba.dubbo.rpc.RpcException: Failed to invoke the method testConcurrent in the service cn.irenshi.dubbo.recruit.ResumeDubboService. Tried 3 times of the providers [192.168.1.148:20880] (1/1) from the registry 192.168.1.2:2181 on the consumer 10.0.0.101 using the dubbo version 2.5.3. Last error is: Failed to invoke remote method: testConcurrent, provider: dubbo://192.168.1.148:20880/cn.irenshi.dubbo.recruit.ResumeDubboService?anyhost=true&application=irenshi-agent-manage-consumer&check=false&default.check=false&default.service.filter=-exception&default.timeout=86400000&dubbo=2.5.3&executes=3&interface=cn.irenshi.dubbo.recruit.ResumeDubboService&methods=findOne,testConcurrent,setResumeReceiveEmailConnect,findAllResume,receiveResume,receiveAndSaveResume&owner=invalid&payload=1073741824&pid=8089&revision=1.1.0-SNAPSHOT&side=consumer&timestamp=1470818605751&version=1.2, cause: com.alibaba.dubbo.rpc.RpcException: Failed to invoke method testConcurrent in provider dubbo://192.168.1.148:20880/cn.irenshi.dubbo.recruit.ResumeDubboService?anyhost=true&application=irenshi-dubbo-provider&default.service.filter=-exception&dubbo=2.5.3&executes=3&interface=cn.irenshi.dubbo.recruit.ResumeDubboService&methods=findOne,testConcurrent,setResumeReceiveEmailConnect,findAllResume,receiveResume,receiveAndSaveResume&owner=invalid&payload=1073741824&pid=7933&revision=1.1.0-SNAPSHOT&side=provider&timestamp=1470818565576&version=1.2, cause: The service using threads greater than <dubbo:service executes="3" /> limited.
com.alibaba.dubbo.rpc.RpcException: Failed to invoke method testConcurrent in provider dubbo://192.168.1.148:20880/cn.irenshi.dubbo.recruit.ResumeDubboService?anyhost=true&application=irenshi-dubbo-provider&default.service.filter=-exception&dubbo=2.5.3&executes=3&interface=cn.irenshi.dubbo.recruit.ResumeDubboService&methods=findOne,testConcurrent,setResumeReceiveEmailConnect,findAllResume,receiveResume,receiveAndSaveResume&owner=invalid&payload=1073741824&pid=7933&revision=1.1.0-SNAPSHOT&side=provider&timestamp=1470818565576&version=1.2, cause: The service using threads greater than <dubbo:service executes="3" /> limited.
	at com.alibaba.dubbo.rpc.filter.ExecuteLimitFilter.invoke(ExecuteLimitFilter.java:43)
	at com.alibaba.dubbo.rpc.protocol.ProtocolFilterWrapper$1.invoke(ProtocolFilterWrapper.java:91)
	at com.alibaba.dubbo.rpc.filter.ContextFilter.invoke(ContextFilter.java:60)
	at com.alibaba.dubbo.rpc.protocol.ProtocolFilterWrapper$1.invoke(ProtocolFilterWrapper.java:91)
	at com.alibaba.dubbo.rpc.filter.GenericFilter.invoke(GenericFilter.java:112)
	at com.alibaba.dubbo.rpc.protocol.ProtocolFilterWrapper$1.invoke(ProtocolFilterWrapper.java:91)
	at com.alibaba.dubbo.rpc.filter.ClassLoaderFilter.invoke(ClassLoaderFilter.java:38)
	at com.alibaba.dubbo.rpc.protocol.ProtocolFilterWrapper$1.invoke(ProtocolFilterWrapper.java:91)
	at com.alibaba.dubbo.rpc.filter.EchoFilter.invoke(EchoFilter.java:38)
	at com.alibaba.dubbo.rpc.protocol.ProtocolFilterWrapper$1.invoke(ProtocolFilterWrapper.java:91)
	at com.alibaba.dubbo.rpc.protocol.dubbo.DubboProtocol$1.reply(DubboProtocol.java:108)
	at ...
Caused by: com.alibaba.dubbo.remoting.RemotingException: com.alibaba.dubbo.rpc.RpcException: Failed to invoke method testConcurrent in provider dubbo://192.168.1.148:20880/cn.irenshi.dubbo.recruit.ResumeDubboService?anyhost=true&application=irenshi-dubbo-provider&default.service.filter=-exception&dubbo=2.5.3&executes=3&interface=cn.irenshi.dubbo.recruit.ResumeDubboService&methods=findOne,testConcurrent,setResumeReceiveEmailConnect,findAllResume,receiveResume,receiveAndSaveResume&owner=invalid&payload=1073741824&pid=7933&revision=1.1.0-SNAPSHOT&side=provider&timestamp=1470818565576&version=1.2, cause: The service using threads greater than <dubbo:service executes="3" /> limited.
com.alibaba.dubbo.rpc.RpcException: Failed to invoke method testConcurrent in provider dubbo://192.168.1.148:20880/cn.irenshi.dubbo.recruit.ResumeDubboService?anyhost=true&application=irenshi-dubbo-provider&default.service.filter=-exception&dubbo=2.5.3&executes=3&interface=cn.irenshi.dubbo.recruit.ResumeDubboService&methods=findOne,testConcurrent,setResumeReceiveEmailConnect,findAllResume,receiveResume,receiveAndSaveResume&owner=invalid&payload=1073741824&pid=7933&revision=1.1.0-SNAPSHOT&side=provider&timestamp=1470818565576&version=1.2, cause: The service using threads greater than <dubbo:service executes="3" /> limited.
	at com.alibaba.dubbo.rpc.filter.ExecuteLimitFilter.invoke(ExecuteLimitFilter.java:43)
	at com.alibaba.dubbo.rpc.protocol.ProtocolFilterWrapper$1.invoke(ProtocolFilterWrapper.java:91)
	at com.alibaba.dubbo.rpc.filter.ContextFilter.invoke(ContextFilter.java:60)
	at com.alibaba.dubbo.rpc.protocol.ProtocolFilterWrapper$1.invoke(ProtocolFilterWrapper.java:91)
	at com.alibaba.dubbo.rpc.filter.GenericFilter.invoke(GenericFilter.java:112)
	at com.alibaba.dubbo.rpc.protocol.ProtocolFilterWrapper$1.invoke(ProtocolFilterWrapper.java:91)
	at com.alibaba.dubbo.rpc.filter.ClassLoaderFilter.invoke(ClassLoaderFilter.java:38)
	at com.alibaba.dubbo.rpc.protocol.ProtocolFilterWrapper$1.invoke(ProtocolFilterWrapper.java:91)
	at com.alibaba.dubbo.rpc.filter.EchoFilter.invoke(EchoFilter.java:38)
	at com.alibaba.dubbo.rpc.protocol.ProtocolFilterWrapper$1.invoke(ProtocolFilterWrapper.java:91)
	at com.alibaba.dubbo.rpc.protocol.dubbo.DubboProtocol$1.reply(DubboProtocol.java:108)
	at ...
```

### `executes`和`actives`参数

Dubbo服务器端参数：
```java
@Service(version = "1.2", executes = 3, actives = 3)
```

