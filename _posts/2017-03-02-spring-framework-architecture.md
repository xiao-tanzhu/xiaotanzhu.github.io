---
layout: post
title: Spring 模块介绍
tags: [Spring]
categories: [Spring]
description: Spring 模块介绍
---

## 前言

Spring Framework 作为一个优秀的开源框架，是为了解决企业应用程序开发复杂性而创建的。框架的主要优势之一就是其分层架构，分层架构允许您选择使用哪一个组件，同时为J2EE应用程序开发提供集成的框架。

## Spring Framework的整体架构

Spring Framework总共有十几个组件，其中核心组件只有三个：Core、Context和Beans。

### Spring Framework 3.x 的总体架构图

![Spring 3.x总体架构图](/upload/images/14.png)

组成 Spring Framework的每个模块（或组件）都可以单独存在，或者与其他一个或多个模块联合实现。每个模块的功能如下：

- **Spring Core（核心容器）：** 核心容器提供 Spring 框架的基本功能。核心容器的主要组件是 BeanFactory，它是工厂模式的实现。BeanFactory 使用控制反转（IOC）模式将应用程序的配置和依赖性规范与实际的应用程序代码分开。

- **Spring Context（上下文）：** Spring 上下文是一个配置文件，向 Spring 框架提供上下文信息。Spring 上下文包括企业服务，例如：JNDI、EJB、电子邮件、国际化、校验和调度功能。

- **Spring AOP：** 通过配置管理特性，Spring AOP 模块直接将面向方面的编程功能集成到了 Spring 框架中。所以，可以很容易地使 Spring 框架管理的任何对象支持 AOP。Spring AOP 模块为基于 Spring 的应用程序中的对象提供了事务管理服务。通过使用 Spring AOP，不用依赖 EJB 组件，就可以将声明性事务管理集成到应用程序中。

- **Spring DAO：** JDBC DAO 抽象层提供了有意义的异常层次结构，可用该结构来管理异常处理和不同数据库供应商抛出的错误消息。异常层次结构简化了错误处理，并且极大地降低了需要编写的异常代码数量（例如打开和关闭连接）。Spring DAO 的面向 JDBC 的异常遵从通用的 DAO 异常层次结构。

- **Spring ORM：** Spring 框架插入了若干个 ORM 框架，从而提供了 ORM 的对象关系工具，其中包括 JDO、Hibernate 和 iBatis SQL Map。所有这些都遵从 Spring 的通用事务和 DAO 异常层次结构。

- **Spring Web 模块：** Web 上下文模块建立在应用程序上下文模块之上，为基于 Web 的应用程序提供了上下文。所以，Spring 框架支持与 Jakarta Struts 的集成。Web 模块还简化了处理多部分请求以及将请求参数绑定到域对象的工作。

- **Spring MVC 框架：** MVC 框架是一个全功能的构建 Web 应用程序的 MVC 实现。通过策略接口，MVC 框架变成为高度可配置的，MVC 容纳了大量视图技术，其中包括 JSP、Velocity、Tiles、iText 和 POI。

从图中可以看出，IOC 的实现包 spring-beans 和 AOP 的实现包 spring-aop 也是整个框架的基础，而 spring-core 是整个框架的核心，基础的功能都在这里。

在此基础之上，spring-context 提供上下文环境，为各个模块提供粘合作用。

在 spring-context 基础之上提供了 spring-tx 和 spring-orm包，而web部分的功能，都是要依赖spring-web来实现的。

### Spring Framework 4.x 的系统架构图

![Spring Framework 4.x 的系统架构图](/upload/images/15.png)

**Spring Framework 4.x对比Spring Framework 3.2.x的系统架构变化：**

  1. Spring 4.0.3去掉了 struts 模块(spring-struts包)

  现在的 Spring mvc的确已经足够优秀了，大量的 web 应用均已经使用了 Spring mvc。而 struts1.x 的架构太落后了，struts2.x 是 struts 自身提供了和 Spring 的集成包，但是由于之前版本的 struts2 存在很多致命的安全漏洞，所以，大大影响了其使用度，好在最新的2.3.16版本的 struts 安全有所改善，希望不会再出什么大乱子。

  1. 增加 WebSocket 模块(spring-websocket包)

  增加了对 WebSocket、SockJS 以及 STOMP 的支持，它与 JSR-356 Java WebSocket API 兼容。另外，还提供了基于 SockJS（对 WebSocket 的模拟）的回调方案，以适应不支持 WebSocket 协议的浏览器。

  1. 增加了 messaging 模块(spring-messaging)

  提供了对 STOMP 的支持，以及用于路由和处理来自 WebSocket 客户端的 STOMP 消息的注解编程模型。spring-messaging 模块中还 包含了 Spring Integration 项目中的核心抽象类，如 Message、MessageChannel、MessageHandler。

  1. 加强了 beans 模块，增加spring-beans-groovy

  应用可以部分或完全使用 Groovy 编写。借助于 Spring 4.0，能够使用 Groovy DSL 定义外部的 Bean 配置，这类似于 XML Bean 声明，但是语法更为简洁。使用Groovy还能够在启动代码中直接嵌入Bean的声明。
