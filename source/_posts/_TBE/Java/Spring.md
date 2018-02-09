---
title: Spring
date: 2017-01-01 09:00:00
tags: [Java]
---

整理自《Spring实战（第4版）》

<!-- more -->

# 第1章 Spring 之旅

## 容器

**Spring 自带了几种容器实现，可归为两种类型：**

1. bean 工厂
bean factories，由 `org.springframework.beans.factory.BeanFactory` 接口定义。
2. 应用上下文
application 由 `org.springframework.context.ApplicationContext` 接口定义，基于 BeanFactory 之上构建，并提供面向应用的服务。

**bean 工厂太低级，通常选用应用上下文。**

**几种最常用的应用上下文：**

- `AnnotationConfigApplicationContext` 从一个或多个基于 Java 的配置类中加载 Spring 应用上下文。
- `AnnotationConfigWebApplicationContext` 从一个或多个基于 Java 的配置类中加载 Spring Web 应用上下文。
- `ClassPathXmlApplicationContext` 从类路径下的一个或多个 XML 配置文件中加载上下文定义，把应用上下文定义文件当作类资源。
- `FileSystemXmlApplicationContext` 读取文件系统下的一个或多个 XML 配置文件并加载上下文定义。
- `XmlWebApplicationContext` 读取 Web 应用下的一个或多个 XML 配置文件并装载上下文定义。

## bean 的生命周期

1. Spring 对 bean 进行实例化。
2. Spring 将值和 bean 的引用注入进 bean 对应的属性中。
3. 若 bean 实现了 `BeanNameAware` 接口，Spring 将调用 `setBeanName()` 接口方法。
4. 若 bean 实现了 `BeanFactoryAware` 接口，Spring 将调用 `setBeanFactory()` 接口方法，将 BeanFactory 容器实例传入。
5. 若 bean 实现了 `ApplicationContextAware` 接口，Spring 将调用 `setApplicationContext()` 接口方法，将应用上下文的引用传入。
6. 若 bean 实现了 `BeanPostProcessor` 接口，Spring 将调用它们的 `postProcessBeforeInitialization()` 方法。
7. 若 bean 实现了 `InitializingBean` 接口，Spring 将调用它们的 `afterPropertiesSet()` 方法。类似地，如果 bean 使用 `init-method` 声明了初始化方法，该方法也会被调用。
8. 若 bean 实现了 `BeanPostProcessor` 接口，Spring 将调用它们的 `postProcessAfterInitialization()` 方法。
9. 此时，bean 已经准备就绪，将一直驻留在应用上下文中，直到该应用上下文被销毁。
10. 若 bean 实现了 `DisposableBean` 接口，Spring 将调用它的 `destroy()` 接口方法。同样，若 bean 使用 `destroy-method` 声明了销毁方法，该方法也会被调用。

## Spring 模块

**模块分类**

- Spring 核心容器
	- Beans
	- Core
	- Context
	- Expression
	- Context support
- 数据访问与集成
	- JDBC
	- Transaction
	- ORM
	- OXM
	- Messaging
	- JMS
- Web 与远程调用
	- Web
	- Web servlet
	- Web portlet
	- WebSocket
- 面向切面编程
	- AOP
	- Aspects
- Instrumentation
	- Instrument
	- Instrument Tomcat
- 测试
	- Test

1. Spring 核心容器
容器是 Spring 框架最核心的部分，管理着 Spring 应用中 bean 的创建、配置和管理。除了 bean 工厂和应用上下文，也提供了许多企业服务，如 E-mail、JNDI 访问、EJB 集成和调度。
2. Spring 的 AOP 模块
Spring 应用系统中开发切面的基础。
3. 数据访问与集成
4. Web 与远程调用
自带一个强大的 MVC 框架；提供多种远程调用方案。
5. Instrumentation
提供了为 JVM 添加代理（agent）的功能。
6. 测试

## Spring 的新功能

**3.1**

3.1 很大程度上聚焦于配置改善和其他一些增强。

- 为了解决各种环境下选择不同配置的问题，引入了环境 profile 功能，从而能根据部署环境来自动切换不同的数据源 bean；
- 在 3.0 基于 Java 的配置之上，3.1 添加了多个 enable 注解，这样就能使用这个注解启用 Spring 的特定功能；
- 添加了对声明式缓存的支持；
- 新添加的用于构造器注入的 c 命名空间，类似于 2.0 所提供的面向属性的 p 命名空间；
- 开始支持 Servlet 3.0，包括在基于 Java 的配置中声明 Servlet 和 Filter，而不再借助于 web.xml；
- 改善对 JPA 的支持，不必再使用 persistence.xml 文件。

以及针对 Spring MVC 的功能增强

- 自动绑定路径变量到模型属性中；
- 提供了 `@RequestMappingproduces` 和 `consumes` 属性，用于匹配请求中的 `Accept` 和 `Content-Type` 头部消息；
- 提供了 `@RequestPart` 注解，用于将 multipart 请求中的某些部分绑定到处理器的方法参数中；
- 支持 flash 属性（在 redirect 请求之后依然能够存活的属性）以及用于在请求间存放 flash 属性的 `RedirectAttributes` 类型。

废弃的功能

- 为了支持原生的 EntityManager，`JpaTemplate` 和 `JpaDaoSupport` 类被废弃掉了。直到 3.2 版本依然可用，4.0 版本中移除。

**3.2**

3.2 主要关注 Spring MVC 的一个发布版本。

- 3.2 的控制器（Controller）可以使用 Servlet 3.0 的异步请求，允许在一个独立的线程中处理请求，从而将 Servlet 线程解放出来处理更多的请求；
- 引入了 Spring MVC 测试框架，用于为控制器编写更为丰富的测试，而且在使用的过程中不需要 Servlet 容器；
- 包含了基于 `RestTemplate` 的客户端的测试支持，测试过程中不需要往真正的 REST 端点上发送请求；
- `ControllerAdvice` 注解能将通用的 `@ExceptionHandler`、`@InitBinder` 和 `@ModelAttributes` 方法收集到一个类中，并应用到所有控制器上；
- 3.2 之前，只能通过 `ContentNegotiatingViewResolver` 使用完整的内容协商（full content negotiating）功能。但在 3.2 中，完整的内容协商功能可以在整个 Spring MVC 中使用，即便是依赖于消息转换器使用和产生内容的控制器方法也能使用该功能；
- 3.2 包含了一个新的 `@MatrixVariable` 注解，能够将请求中的矩阵变量绑定到处理器的方法参数中；
- 基础的抽象类 `AbstractDispatcherServletInitializer` 能够非常便利地配置 `DispatcherServlet`，而不必再使用 web.xml。与之类似，当希望通过基于 Java 的方式来配置 Spring 时，可以使用 `AbstractAnnotationConfigDispatcherServletInitializer` 的子类；
- 新增了 `ResponseEntityExceptionHandler`，可以用来替代 `DefaultHandlerException Resolver`。`ResponseEntityExceptionHandler` 方法会返回 `ResponseEntity<Object>`，而不是 `ModelAndView`；
- `RestTemplate` 和 `@RequestBody` 的参数可以支持泛型；
- `RestTemplate` 和 `@RequestMapping` 可以支持 HTTP `PATCH` 方法；
- 在拦截器匹配时，支持使用 URL 模式将其排除在拦截器的处理功能之外。

3.2 也增加了多项非 MVC 的功能改善。

- `@Autowired`、`@Value` 和 `@Bean` 注解能够作为元注解，用于创建自定义的注入和 bean 声明注解；
- `@DateTimeFormat` 注解不再强依赖 JodaTime。如果提供了 JodaTime，就会使用它，否则会使用 SimpleDateFormat；
- Spring 的声明式缓存提供了对 JCache 0.5 的支持；
- 支持定义全局的格式来解析和渲染日期与时间；
- 在集成测试中，能够配置和加载 `WebApplicationContext`；
- 在集成测试中，能够针对 request 和 session 作用域的 bean 进行测试。

**4.0**

- 对 WebSocket 编程的支持，包括支持 JSR-356——Java API for WebSocket；
- 鉴于 WebSocket 仅仅提供了一种低层次的 API，急需高层次的抽象，因此 4.0 在 WebSocket 之上提供了一个高层次的面向消息的编程类型，该模型基于 SockJS，并且包含了对 STOMP 协议的支持；
- 新的消息（messaging）模块，很多的类型来源于 Spring Integration 项目。这个消息模块支持 Spring 的 SockJS/STOMP 功能，同时提供了基于模板的方式发布消息；
- 支持 Java 8 特性；
- 支持 JSR-310——Date 与 Time API，比 `java.util.Date` 或 `java.util.Calendar` 更丰富的 API；
- 为 Groovy 开发的应用程序提供了更加顺畅的编程体验；
- 添加了条件化创建 bean 的功能；
- 包含了 `RestTemplate` 的一个新的异步实现；
- 添加了对多项 JEE 规范的支持，包括 JMS 2.0、JTA 1.2、JPA 2.1 和 Bean Validation 1.1。

# 第2章 装配 bean

## 配置的可选方案

3 种主要的装配机制：

- 在 XML 中进行显式配置；
- 在 Java 中进行显式配置；
- 隐式的 bean 发现机制和自动装配。

## 自动化装配 bean

**创建可被发现的 bean**

```java
package com.demo;

@Component("myFirstClass") // 不指定 ID 的话默认会将类名首字母小写作为 ID
//@Named("myFirstClass") // Spring 支持 Java 依赖注入规范，大多数场景中两者可互相替换
public class MyClass {
}
```

**启用组件扫描**

基于 Java 的配置

```java
package com.demo;

@Configuration
@ComponentScan(basePackages="com.demo") // 不指定基础包的话会以此配置类所在包为基础包
//@ComponentScan(basePackages={"com.demo","com.demo1"}) // 支持指定多个基础包
//@ComponentScan(basePackageClasses={MyClass.class, MyClass1.class})
// 通过类型安全的方式指定基础包，即指定类所在的包。
// 推荐在基础包中创建一个用来进行扫描的空标记接口，方便在修改业务代码的时候不影响组件扫描
public class MySpringConfig {
}
```

如果没有其他配置的话，`@ComponentScan` 默认会扫描与配置类相同的包以及子包，查找带有 `@Component` 注解的类。

基于 XML 的配置

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:Context="http://www.springframework.org/schema/context"
	xsi:schemaLocation="http://www.springframework.org/schema/beans
		http://www.springframework.org/schema/beans/spring-beans.xsd
		http://www.springframework.org/schema/context
		http://www.springframework.org/schema/context/spring-context.xsd">

	<context:component-scan base-package="com.demo"/>

</beans>
```

**通过注解实现自动装配**

在构造方法、setter 方法或者其他任何方法上使用 `@Autowired`，Spring 会根据方法参数类型传入适当的 bean。

若没有匹配的 bean，Spring 会抛出一个异常。可将 `@Autowired` 的 `required` 属性设置为 `false`，从而在匹配不到的时候传入 null，且避免抛出异常。

可替换为 Java 依赖注入规范中的 `@Inject`。

## 通过 Java 代码装配 bean

**创建配置类**

```java
package com.demo;

@Configuration
public class MyConfig {
}
```

**声明 bean**

```java
@Bean
public MyClass myClass() {
	return new MyClass();
}
```

`@Bean` 告诉 Spring 将该方法返回的对象注册为 bean，ID 默认为方法名，也可指定 ID：`@Bean(name="myClass")`。

**借助 JavaConfig 实现注入**

```java
@Bean
public MyOutterClass myOutterClass() {
	return new MyOutterClass(myClass())
}

这种通过调用方法来引用 bean 的方式有时候可能引起误解，让人以为每次都拿到一个新的对象，而实际上 Spring 的 bean 默认是单例的。

可以换一种方式：

```java
@Bean
public MyOutterClass myOutterClass(MyClass myClass) {
	return new MyOutterClass(myClass);
}
```

这种方式一方面不易引起误解，另一方面允许 `myClass` 声明于其他地方。

## 通过 XML 装配 bean**

**创建 XML 配置规范**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://www.springframework.org/schema/beans
		http://www.springframework.org/schema/beans/spring-beans.xsd
		http://www.springframework.org/schema/context">

	<!-- configuration details -->

</beans>
```

**声明 <bean>**

```xml
<bean id="myClass" class="com.demo.MyClass"/>
```

若不指明 ID，则默认为类名首字母小写+“#0”。后续声明的同类型实例就是类名首字母小写+“#1”。

**构造器注入 bean**

```xml
<bean id="myOutterClass" class="com.demo">
	<constructor-arg ref="myClass"/>
</bean>
```

或使用 c- 命名空间（3.0 中引入）。
首先在顶部声明命名空间模式。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:c="http://www.springframework.org/schema/c"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://www.springframework.org/schema/beans
		http://www.springframework.org/schema/beans/spring-beans.xsd">

	<bean id="myOutterClass" class="com.demo.MyOutterClass"
		c:myClass-ref="myClass">

</beans>
```

该属性名的结构：

- `c:` c-命名空间前缀
- `myClass` 构造器参数名
- `-ref` 注入 bean 引用。去掉此项则注入字面量。
- `"myClass"` 要注入的 bean ID

由于引用了构造器参数名，所以如果移除构建过程中的调试标志（debug symbol）就无法正常执行了。

替代方案是使用参数在参数列表中的位置信息：

```xml
<bean id="myOutterClass" class="com.demo.MyOutterClass"
		c:_0-ref="myClass">
```

因为 XML 中不允许数字作为属性的第一个字符，所以要添加一个下划线。

若只有一个构造器参数，则可以不用标示参数：

```xml
<bean id="myOutterClass" class="com.demo.MyOutterClass"
		c:_-ref="myClass">
```

**构造器注入字面量**

```xml
<bean id="myClass" class="com.demo.MyClass">
	<constructor-arg value="abc" />
</bean>
```

**构造器装配集合**


# 第 9 章 保护 Web 应用

## Spring Security 的模块

|模块                    |描述                                                                                                 |
|:-----------------------|:----------------------------------------------------------------------------------------------------|
|ACL                     |支持通过访问控制列表（access control list,ACL）为域对象提供安全性                                    |
|切面（Aspects）         |一个很小的模块，当使用 Spring Security 注解时，会使用基于 AspectJ 的切面，而不是使用标准的 Spring AOP|
|CAS 客户端（CAS Client）|提供与 Jasig 的中心认证服务（Central Authentication Service, CAS）进行集成的功能                     |
|配置（Configuration）   |包含通过 XML 和 Java 配置 Spring Security 的功能支持                                                 |
|核心（core）            |提供 Spring Security 基本库                                                                          |
|加密（Cryptography）    |提供了加密和密码编码功能                                                                             |
|LDAP                    |支持基于 LDAP 进行认证                                                                               |
|OpenID                  |支持使用 OpenID 进行集中式认证                                                                       |
|Remoting                |提供了对 Spring Remoting 的支持                                                                      |
|标签库（Tag Library）   |Spring Security 的 JSP 标签库                                                                        |
|Web                     |提供了 Spring Security 基于 Filter 的 Web 安全性支持                                                 |
