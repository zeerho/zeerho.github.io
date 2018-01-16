---
title: Spring
date: 2017-01-01 09:00:00
tags: [Java]
---

整理自《Spring实战（第3版）》

# 第1章 Spring 之旅

**Spring 自带了几种容器实现，可归为两种类型：**

1. Bean 工厂
bean factories，由 `org.springframework.beans.factory.BeanFactory` 接口定义。
2. 应用上下文
application 由 `org.springframework.context.ApplicationContext` 接口定义，基于 BeanFactory 之上构建，并提供面向应用的服务。

**Bean工厂太低级，通常选用应用上下文。**

**3 种最常用的应用上下文：**

- `ClassPathXmlApplicationContext` 从类路径下的 XML 配置文件中加载上下文定义，把应用上下文定义文件当作类资源。
- `FileSystemXmlApplicationContext` 读取文件系统下的 XML 配置文件并加载上下文定义。
- `XmlWebApplicationContext` 读取 Web 应用下的 XML 配置文件并装载上下文定义。

**Bean 的生命周期**
1. Spring 对 Bean 进行实例化。
2. Spring 将值和 Bean 的引用注入进 Bean 对应的属性中。
3. 若 Bean 实现了 `BeanFactoryAware` 接口，Spring 将调用 `setBeanName()` 接口方法。
4. 若 Bean 实现了 `BeanFactoryAware` 接口，Spring 将调用 `setBeanFactory()` 接口方法，将 BeanFactory 容器实例传入。
5. 若 Bean 实现了 `ApplicationContextAware` 接口，Spring 将调用 `setApplicationContext()` 接口方法，将应用上下文的引用传入。

