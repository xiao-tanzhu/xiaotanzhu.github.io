---
layout: post
title: "Spring Framework: Modules"
tags: [Spring, "Spring Modules"]
categories: [Spring]
description: "Spring Framework: Modules"
---

> Reference: https://docs.spring.io/spring/docs/4.3.0.BUILD-SNAPSHOT/spring-framework-reference/htmlsingle/#overview-modules

## Spring Framework: Modules

The Spring Framework consists of features organized into about 20 modules. These modules are grouped into `Core Container`, `Data Access/Integration`, `Web`, `AOP (Aspect Oriented Programming)`, `Instrumentation`, `Messaging`, and `Test`, as shown in the following diagram.

![Spring Framework 4.x 的系统架构图](/upload/images/15.png)

The following sections list the available modules for each feature along with their artifact names and the topics they cover.

### 1. Core Container

> - spring-core
> - spring-beans
> - spring-context
> - spring-context-support
> - spring-expression

The *Core Container* consists of the `spring-core`, `spring-beans`, `spring-context`, `spring-context-support`, and `spring-expression` (Spring Expression Language) modules.

The `spring-core` and `spring-beans` modules provide the fundamental parts of the framework, including the *IoC* and *Dependency Injection* features. The `BeanFactory` is a sophisticated implementation of the factory pattern. It removes the need for programmatic singletons and allows you to decouple the configuration and specification of dependencies from your actual program logic.

The Context (`spring-context`) module builds on the solid base provided by the *Core* and *Beans* modules: it is a means to access objects in a framework-style manner that is similar to a JNDI registry. The Context module inherits its features from the *Beans* module and adds support for `internationalization` (using, for example, resource bundles), `event propagation`, `resource loading`, and `the transparent creation of contexts` by, for example, a Servlet container. The Context module also supports Java EE features such as EJB, JMX, and basic remoting. The ApplicationContext interface is the focal point of the Context module. spring-context-support provides support for integrating common third-party libraries into a Spring application context for caching (EhCache, Guava, JCache), mailing (JavaMail), scheduling (CommonJ, Quartz) and template engines (FreeMarker, JasperReports, Velocity).

The `spring-expression` module provides a powerful Expression Language for querying and manipulating an object graph at runtime. It is an extension of the unified expression language (unified EL) as specified in the JSP 2.1 specification. The language supports setting and getting property values, property assignment, method invocation, accessing the content of arrays, collections and indexers, logical and arithmetic operators, named variables, and retrieval of objects by name from Spring’s IoC container. It also supports list projection and selection as well as common list aggregations.

### 2. AOP and Instrumentation

> - spring-aop
> - spring-aspect
> - spring-instrument
> - spring-instrument-tomcat

The `spring-aop` module provides an AOP Alliance-compliant aspect-oriented programming implementation allowing you to define, for example, method interceptors and pointcuts to cleanly decouple code that implements functionality that should be separated. Using source-level metadata functionality, you can also incorporate behavioral information into your code, in a manner similar to that of .NET attributes.

The separate `spring-aspects` module provides integration with AspectJ.

The `spring-instrument` module provides class instrumentation support and classloader implementations to be used in certain application servers. The `spring-instrument-tomcat` module contains Spring’s instrumentation agent for Tomcat.

### 3. Messaging

> - spring-messaging

Spring Framework 4 includes a `spring-messaging` module with key abstractions from the *Spring Integration* project such as `Message`, `MessageChannel`, `MessageHandler`, and others to serve as a foundation for messaging-based applications. The module also includes a set of annotations for mapping messages to methods, similar to the Spring MVC annotation based programming model.

### 4. Data Access/Integration

> - spring-jdbc
> - spring-tx
> - spring-orm
> - spring-oxm
> - spring-jms

The *Data Access/Integration layer* consists of the JDBC, ORM, OXM, JMS, and Transaction modules.

The `spring-jdbc` module provides a JDBC-abstraction layer that removes the need to do tedious JDBC coding and parsing of database-vendor specific error codes.

The `spring-tx` module supports programmatic and declarative transaction management for classes that implement special interfaces and for all your POJOs (Plain Old Java Objects).

The `spring-orm` module provides integration layers for popular object-relational mapping APIs, including JPA, JDO, and Hibernate. Using the `spring-orm` module you can use all of these O/R-mapping frameworks in combination with all of the other features Spring offers, such as the simple declarative transaction management feature mentioned previously.

The `spring-oxm` module provides an abstraction layer that supports Object/XML mapping implementations such as JAXB, Castor, XMLBeans, JiBX and XStream.

The `spring-jms` module (Java Messaging Service) contains features for producing and consuming messages. Since Spring Framework 4.1, it provides integration with the `spring-messaging` module.

### 5. Web

> - spring-web
> - spring-webmvc: Web-Servlet module
> - spring-webmvc-portlet: Web-Portlet module

The Web layer consists of the `spring-web`, `spring-webmvc`, `spring-websocket`, and `spring-webmvc-portlet` modules.

The `spring-web` module provides basic web-oriented integration features such as multipart file upload functionality and the initialization of the IoC container using Servlet listeners and a web-oriented application context. It also contains an HTTP client and the web-related parts of Spring’s remoting support.

The `spring-webmvc` module (also known as the Web-Servlet module) contains Spring’s model-view-controller (MVC) and REST Web Services implementation for web applications. Spring’s MVC framework provides a clean separation between domain model code and web forms and integrates with all of the other features of the Spring Framework.

The `spring-webmvc-portlet` module (also known as the Web-Portlet module) provides the MVC implementation to be used in a Portlet environment and mirrors the functionality of the spring-webmvc module.

### 6. Test

> - spring-test

The `spring-test` module supports the unit testing and integration testing of Spring components with JUnit or TestNG. It provides consistent loading of Spring ApplicationContexts and caching of those contexts. It also provides mock objects that you can use to test your code in isolation.
