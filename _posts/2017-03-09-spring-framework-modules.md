---
layout: post
title: "Spring Framework: Modules"
tags: [Spring, "Spring Modules"]
categories: [Spring]
description: Spring Framework: Modules
---

> Reference: https://docs.spring.io/spring/docs/4.3.0.BUILD-SNAPSHOT/spring-framework-reference/htmlsingle/#overview-modules

## Spring Framework: Modules

The Spring Framework consists of features organized into about 20 modules. These modules are grouped into `Core Container`, `Data Access/Integration`, `Web`, `AOP (Aspect Oriented Programming)`, `Instrumentation`, `Messaging`, and `Test`, as shown in the following diagram.

![Spring Framework 4.x 的系统架构图](/upload/images/15.png)

The following sections list the available modules for each feature along with their artifact names and the topics they cover.

### 1. Core Container

The *Core Container* consists of the `spring-core`, `spring-beans`, `spring-context`, `spring-context-support`, and `spring-expression` (Spring Expression Language) modules.

The `spring-core` and `spring-beans` modules provide the fundamental parts of the framework, including the *IoC* and *Dependency Injection* features. The `BeanFactory` is a sophisticated implementation of the factory pattern. It removes the need for programmatic singletons and allows you to decouple the configuration and specification of dependencies from your actual program logic.

The Context (`spring-context`) module builds on the solid base provided by the *Core* and *Beans* modules: it is a means to access objects in a framework-style manner that is similar to a JNDI registry. The Context module inherits its features from the Beans module and adds support for internationalization (using, for example, resource bundles), event propagation, resource loading, and the transparent creation of contexts by, for example, a Servlet container. The Context module also supports Java EE features such as EJB, JMX, and basic remoting. The ApplicationContext interface is the focal point of the Context module. spring-context-support provides support for integrating common third-party libraries into a Spring application context for caching (EhCache, Guava, JCache), mailing (JavaMail), scheduling (CommonJ, Quartz) and template engines (FreeMarker, JasperReports, Velocity).

The spring-expression module provides a powerful Expression Language for querying and manipulating an object graph at runtime. It is an extension of the unified expression language (unified EL) as specified in the JSP 2.1 specification. The language supports setting and getting property values, property assignment, method invocation, accessing the content of arrays, collections and indexers, logical and arithmetic operators, named variables, and retrieval of objects by name from Spring’s IoC container. It also supports list projection and selection as well as common list aggregations.
