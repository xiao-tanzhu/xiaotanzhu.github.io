---
layout: post
title: "Spring Cloud组件介绍"
tags: ["微服务", "Spring Cloud"]
categories: [Spring]
description: "Spring Cloud组件介绍"
---

> Reference:
> - [Spring Cloud全家桶主要组件及简要介绍](https://blog.csdn.net/xlgen157387/article/details/77773908)
> - [SpringCloud分布式开发五大组件详解](https://blog.csdn.net/wxb880114/article/details/79467779)
> - [配置中心和消息总线（配置中心终结版）](http://www.ityouknow.com/springcloud/2017/05/26/springcloud-config-eureka-bus.html)

## 什么是微服务？

微服务的主旨是将一个原本独立的系统拆分成多个小型服务，这些小型服务都在各自独立的进程中运行，服务之间通过基于HTTP的RESTful API进行通信协作，并且每个服务都维护着自身的数据存储、业务开发、自动化测试以及独立部署机制。

一个微服务一般完成某个特定的功能，比如下单管理、客户管理等等。每一个微服务都是微型六角形应用，都有自己的业务逻辑和适配器。一些微服务还会发布API给其它微服务和应用客户端使用。其它微服务完成一个Web UI，运行时，每一个实例可能是一个云VM或者是Docker容器。

以下是一个汽车租赁网站的微服务架构：

![微服务示例](/upload/images/16.png)

## 常见微服务框架

### 服务治理框架

1. Dubbo（[http://dubbo.io/](http://dubbo.io/)）、Dubbox（当当网对Dubbo的扩展）
1. Netflix的Eureka、Apache的Consul等。

  > Spring Cloud Eureka是对Netflix的Eureka的进一步封装。

### 分布式配置管理

1. 百度的Disconf
2. 360的QConf
3. Spring Cloud组件中的Config
4. 淘宝的Diamond

### 批量任务框架

1. Spring Cloud组件中的Task
1. LTS

### 服务追踪框架

1. Skywalking

## Spring Cloud全家桶组件

Netflix 公司提供了包括Eureka、Hystrix、Zuul、Archaius等在内的很多组件，在微服务架构中至关重要，Spring在Netflix 的基础上，封装了一系列的组件，命名为：Spring Cloud Eureka、Spring Cloud Hystrix、Spring Cloud Zuul等，下边对各个组件进行分别得介绍：

### Spring Cloud Eureka

我们可以将自己定义的API 接口注册到Spring Cloud Eureka上，Eureka负责服务的注册于发现，如果学习过Zookeeper的话，就可以很好的理解，Eureka的角色和 Zookeeper的角色差不多，都是服务的注册和发现，构成Eureka体系的包括：服务注册中心、服务提供者、服务消费者。

![](/upload/images/17.png)

上图中描述了：

1. 两台Eureka服务注册中心构成的服务注册中心的主从复制集群；
2. 然后服务提供者向注册中心进行注册、续约、下线服务等；
3. 服务消费者向Eureka注册中心拉去服务列表并维护在本地（这也是客户端发现模式的机制体现！）；
4. 然后服务消费者根据从Eureka服务注册中心获取的服务列表选取一个服务提供者进行消费服务。

### Spring Cloud Ribbon

在上Spring Cloud Eureka描述了服务如何进行注册，注册到哪里，服务消费者如何获取服务生产者的服务信息，但是Eureka只是维护了服务生产者、注册中心、服务消费者三者之间的关系，真正的服务消费者调用服务生产者提供的数据是通过Spring Cloud Ribbon来实现的。

![](/upload/images/18.png)

Ribbon，主要提供客户侧的软件负载均衡算法。

Ribbon客户端组件提供一系列完善的配置选项，比如连接超时、重试、重试算法等。Ribbon内置可插拔、可定制的负载均衡组件。下面是用到的一些负载均衡策略：

- 简单轮询负载均衡
- 加权响应时间负载均衡
- 区域感知轮询负载均衡
- 随机负载均衡

Ribbon中还包括以下功能：

- 易于与服务发现组件（比如Netflix的Eureka）集成
- 使用Archaius完成运行时配置
- 使用JMX暴露运维指标，使用Servo发布
- 多种可插拔的序列化选择
- 异步和批处理操作（即将推出）
- 自动SLA框架（即将推出）
- 系统管理/指标控制台（即将推出）


### Spring Cloud Feign

### Spring Cloud Hystrix

当有一个服务出现了故障，而服务的调用方不知道服务出现故障，若此时调用放的请求不断的增加，最后就会等待出现故障的依赖方 相应形成任务的积压，最终导致自身服务的瘫痪。

Spring Cloud Hystrix正是为了解决这种情况的，防止对某一故障服务持续进行访问。Hystrix的含义是：断路器，断路器本身是一种开关装置，用于我们家庭的电路保护，防止电流的过载，当线路中有电器发生短路的时候，断路器能够及时切换故障的电器，防止发生过载、发热甚至起火等严重后果。

![](/upload/images/19.png)

断路器可以防止一个应用程序多次试图执行一个操作，即很可能失败，允许它继续而不等待故障恢复或者浪费 CPU 周期，而它确定该故障是持久的。断路器模式也使应用程序能够检测故障是否已经解决。如果问题似乎已经得到纠正​​，应用程序可以尝试调用操作。

![](/upload/images/20.png)

断路器增加了稳定性和灵活性，以一个系统，提供稳定性，而系统从故障中恢复，并尽量减少此故障的对性能的影响。它可以帮助快速地拒绝对一个操作，即很可能失败，而不是等待操作超时（或者不返回）的请求，以保持系统的响应时间。如果断路器提高每次改变状态的时间的事件，该信息可以被用来监测由断路器保护系统的部件的健康状况，或以提醒管理员当断路器跳闸，以在打开状态。

Hystrix断路器的工作流程：

![](/upload/images/21.png)

### Spring Cloud Zuul

服务网关是微服务架构中一个不可或缺的部分。通过服务网关统一向外系统提供REST API的过程中，除了具备服务路由、均衡负载功能之外，它还具备了权限控制等功能。Spring Cloud Netflix中的Zuul就担任了这样的一个角色，为微服务架构提供了前门保护的作用，同时将权限控制这些较重的非业务逻辑内容迁移到服务路由层面，使得服务集群主体能够具备更高的可复用性和可测试性。

![](/upload/images/22.png)

Zuul类似nginx，反向代理的功能，不过netflix自己增加了一些配合其他组件的特性。

### Spring Cloud Config

对于微服务还不是很多的时候，各种服务的配置管理起来还相对简单，但是当成百上千的微服务节点起来的时候，服务配置的管理变得会复杂起来。

分布式系统中，由于服务数量巨多，为了方便服务配置文件统一管理，实时更新，所以需要分布式配置中心组件。在Spring Cloud中，有分布式配置中心组件Spring Cloud Config ，它支持配置服务放在配置服务的内存中（即本地），也支持放在远程Git仓库中。在Spring Cloud Config 组件中，分两个角色，一是Config Server，二是Config Client。

> - Config Server用于配置属性的存储，存储的位置可以为Git仓库、SVN仓库、本地文件等
> - Config Client用于服务属性的读取

![](/upload/images/23.png)

### Spring Cloud Bus

Spring cloud bus通过轻量消息代理连接各个分布的节点。这会用在广播状态的变化（例如配置变化）或者其他的消息指令。Spring bus的一个核心思想是通过分布式的启动器对spring boot应用进行扩展，也可以用来建立一个多个应用之间的通信频道。目前唯一实现的方式是用AMQP消息代理作为通道，同样特性的设置（有些取决于通道的设置）在更多通道的文档中。

Spring cloud bus被国内很多都翻译为消息总线，也挺形象的。大家可以将它理解为管理和传播所有分布式项目中的消息既可，其实**本质是利用了MQ的广播机制在分布式的系统中传播消息**，目前常用的有Kafka和RabbitMQ。利用bus的机制可以做很多的事情，其中配置中心客户端刷新就是典型的应用场景之一，我们用一张图来描述bus在配置中心使用的机制。

![](/upload/images/25.jpg)

根据此图我们可以看出利用Spring Cloud Bus做配置更新的步骤:

1. 提交代码触发post给客户端A发送bus/refresh
2. 客户端A接收到请求从Server端更新配置并且发送给Spring Cloud Bus
3. Spring Cloud bus接到消息并通知给其它客户端
4. 其它客户端接收到通知，请求Server端获取最新配置
5. 全部客户端均获取到最新的配置
