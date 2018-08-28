---
title: Spring
date: 2017-01-01 09:00:05
tags: [Java, Spring]
---

大部分内容整理自《Spring实战（第4版）》

<!-- more -->

# 第 1 章 Spring 之旅

## xml 扫描的大致流程

**精简版**

1. `BeanDefinitionReader#loadBeanDefinitions`
    1. `DocumentLoader#loadDocument` 得到 `Document` 对象
    2. `BeanDefinitionDocumentReader#registerBeanDefinitions`
        1. 对于每一层 `beans` 标签，都创建一个 `BeanDefinitionParserDelegate`，因而这个 delegate 也可以形成嵌套结构
        2. 解析这层 `beans` 标签下的配置内容
            1. 对于当前命名空间下的元素，直接用当前的 delegate 来解析
            2. 对于其他命名空间下的元素，调用 `NamespaceHanlderResolver#resolve` 来获取命名空间对应的 `NamespaceHandler`。然后调用 `NamespaceHandler#parse` 来解析这个命名空间下的所有元素
               有两种方式来实现命名空间的解析逻辑：
                1. 直接实现 `NamespaceHandler` 接口
                2. 实现 `NamespaceHandlerSupport` 抽象类，在 `init` 方法中注册自定义的 `BeanDefinitionParser` 和 `BeanDefinitionDecorator`，具体的解析逻辑会被委托给这些解析类。命名空间下的每一个元素对应一个解析类。

**详细版**

1. `BeanDefinitionReader#loadBeanDefinitions` 从指定资源或位置加载 bean 定义。`XmlBeanDefinitionReader` 是 xml 对应的实现。
    1. 使用 `DocumentLoader` 把 xml 资源加载成 `org.w3c.dom.Document`。`DocumentLoader` 是加载 xml 的策略接口，默认实现为 `DefaultDocumentLoader`。
    2. 调用 `BeanDefinitionDocumentReader#registerBeanDefinitions` 注册 bean 定义。
        - 调用这个方法的时候，会传入 `XmlReaderContext`，其中包含了 `Resource`、`ProblemReporter`、`ReaderEventListener`、`SourceExtractor`、`XmlBeanDefinitionReader`、`NamespaceHandlerResolver`。
2. `BeanDefinitionDocumentReader` 是用来注册 bean 定义的 SPI，默认实现为 `DefaultBeanDefinitionDocumentReader`。
    1. 调用 `BeanDefinitionParserDelegate#isDefaultNamespace` 判断当前节点是否为默认的命名空间（无命名空间或命名空间为 `beans`），若是的话，判断 `profile` 属性是否符合。
    2. 使用 `BeanDefinitionParserDelegate` 来负责解析 bean 定义。
        - 此类保存了 `beans` 标签中定义的一些属性，供后续的 bean 解析器来使用。
        - 因为 `beans` 标签是可嵌套的，所以 `BeanDefinitionParserDelegate` 也是可嵌套的，对于子节点未配置的属性，缺省为父节点的属性。
        - 此类还对外提供了众多解析用的方法，这些方法一方面提供了统一的 API，另一方面也共用了解析过程中的模板代码。
        - 此类的每一个对象都代表一个命名空间，所以可以继承子类来表示自定义的命名空间，并提供该命名空间下自定义的解析方法。
    3. 递归解析根节点下的配置。
        - 解析默认命名空间下的配置，调用 `BeanDefinitionParserDelegate#parseDefaultElement`。其中包括 `import`、`alias`、`bean`、`beans`，后续可能会扩展。
        - 解析其他命名空间下的配置，调用 `BeanDefinitionParserDelegate#parseCustomElement`。
            1. 取 `XmlReaderContext` 中的 `NamespaceHandlerResolver`，默认实现为 `DefaultNamespaceHandlerResolver`。
                - `NamespaceHandlerResolver` 中维护了一个配置文件的地址（默认为 META-INF/spring.handlers，查找范围包括类加载器下的所有包），该文件中保存了命名空间和 `NamespaceHandler` 实现类的对应关系。
            2. 调用 `NamespaceHandlerResolver#resolve` 找到当前命名空间对应的 `NamespaceHandler`。
            3. 调用 `NamespaceHandler#parse` 来做进一步的解析。
3. `NamespaceHandler` 有一个抽象实现类 `NamespaceHandlerSupport`，一般通过实现此类来自定义一个 `NamespaceHandler`。
    - `NamespaceHandlerSupport` 维护了三个 Map，保存了 `BeanDefinitionParser` / `BeanDefinitionDecorator`，分别用来解析直接挂在 `beans` 下的 `bean`、挂在 `bean` 下的 `bean`、`bean` 中的 attribute;
        - `BeanDefinitionParser`：实际的 bean 解析类，负责解析直接挂在 `beans` 标签下的 bean；
        - `BeanDefinitionDecorator`：实际的 bean 解析类，负责解析挂在 `bean` 标签下的 bean；
    - `NamespaceHandlerSupport` 提供了三个抽象方法，分别用来在上述三个 Map 中注册解析类。
    - 当 `parse` 方法被调用时，根据当前 xml 元素的名称从上述 Map 中找到对应的 `BeanDefinitionParser`，然后调用其 `parse` 方法。
    - `BeanDefinitionParser` 根据自己的需要，可以调用 `BeanDefinitionParserDelegate#decorateBeanDefinitionIfRequired` 方法，此方法又会调用 `NamespaceHandlerResolver#resolve` -> `NamespaceHandler#decorate`。


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

*黑体为类，其余为接口或抽象类。有一部分因多重实现而导致的重复。*

- ApplicationContext
  - ConfigurableApplicationContext, WebApplicationContext
    - ConfigurablePortletApplicationContext
      - **StaticPortletApplicationContext**
      - AbstractRefreshablePortletApplicationContext
        - **XmlPortletApplicationContext**
    - ConfigurableWebApplicationContext
      - **GenericWebApplicationContext**
      - **StaticWebApplicationContext**
      - AbstractRefreshableWebApplicationContext
        - **XmlWebApplicationContext**
        - **GroovyWebApplicationContext**
        - **AnnotationConfigWebApplicationContext**
  - ConfigurableApplicationContext
    - AbstractApplicationContext
      - **GenericApplicationContext**
        - **GenericXmlApplicationContext**
        - **StaticAppliationContext**
          - **StaticPortletApplicationContext**
          - **StaticWebApplicationContext**
        - **GenericWebApplicationContext**
        - **ResourceAdapterApplicationContext**
        - **GenericGroovyApplicationContext**
        - **AnnotationConfigApplicationContext**
      - AbstractRefreshableApplicationContext
        - AbstractRefreshableApplicationContext
          - AbstractXmlApplicationContext
            - **FileSystemXmlApplicationContext**
            - **ClassPathXmlApplicationContext**
          - AbstractRefreshablePortletApplicationContext
            - **XmlPortletApplicationContext**
          - AbstractRefreshableWebApplicationContext
            - **XmlWebApplicationContext**
            - **GroovyWebApplicationContext**
            - **AnnotationConfigWebApplicationContext**
        - **AnnotationConfigWebApplicationContext**
  - WebApplicationContext
    - **StubWebApplicationContext**
    - AbstractRefreshablePortletApplicationContext
      - **XmlPortletApplicationContext**

## bean 的生命周期

实例化过程发生在 BeanFactory 中。

1. **若容器注册了 `InstantiationAwareBeanPostProcessor` 接口，则调用其 `postProcessBeforeInstantiation()` 方法。**
1. Spring 通过构造函数或工厂方法对 bean 进行实例化。
1. **若容器注册了 `InstantiationAwareBeanPostProcessor` 接口，则调用其 `postProcessAfterInstantiation()` 方法。**
4. **准备设置配置的属性，先调用 `InstantiationAwareBeanPostProcessor` 接口的 `postProcessPropertyValues()` 方法。**
2. 根据配置设置属性。
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

# 第 2 章 装配 bean

## 配置的可选方案

3 种主要的装配机制：

- 在 XML 中进行显式配置；
- 在 Java 中进行显式配置；
- 隐式的 bean 发现机制和自动装配。

## 通过 XML 装配 bean

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

若不指明 ID，则默认为类名首字母小写 + “#0”。后续声明的同类型实例就是类名首字母小写 + “#1”。

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

## 通过 Java 代码装配 bean

**创建配置类**

```java
package com.demo;

@Configuration
public class MyConfig {
}
```

在 web 环境下还需要做以下配置来让 spring 知道配置类的位置。

```xml
<context-param>
    <param-name>contextClass</param-name>
    <param-value>org.springframework.web.context.support.AnnotationConfigWebApplicationContext</param-value>
</context-param>
<context-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>com.example.MyConfiguration</param-value>
</context-param>
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
```

这种通过调用方法来引用 bean 的方式有时候可能引起误解，让人以为每次都拿到一个新的对象，而实际上 Spring 的 bean 默认是单例的。

可以换一种方式：

```java
@Bean
public MyOutterClass myOutterClass(MyClass myClass) {
	return new MyOutterClass(myClass);
}
```

这种方式一方面不易引起误解，另一方面允许 `myClass` 声明于其他地方。

## 隐式的 bean 发现机制和自动装配

**创建可被发现的 bean**

```java
package com.demo;

@Component("myFirstClass") // 不指定 ID 的话默认会将类名首字母小写作为 ID
//@Named("myFirstClass") // Spring 支持 Java 依赖注入规范，大多数场景中两者可互相替换
public class MyClass {
}
```

**启用组件扫描**

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

## 导入和混合配置

javaConfig 导入 javaConfig 和 xml

```java
@Configuration
@Import({SubConfig1.class, SubConfig2.class})
@ImportResource("classpath:test.xml")
public class BasicConfig {
}
```

xml 导入 javaConfig 和 xml

```xml
<bean class="me.example.BasicConfig"/>
<import resource="cdplayer-config.xml"/>
```

# 第 3 章 高级装配

## 环境与 profile

```java
@Configuration
@Profile("dev") //作用于类中所有 bean
public class ProfileBeanConfig {
	@Bean
	@Profile("dev") //作用于单个 bean
	public MyBean myBean() {
		return new MyBean();
	}
}
```

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XHLSchema-instance"
	xsi:schemaLocation="
		http://www.springframework.org/schema/beans
		http://www.springframework.org/schema/beans/spring-beans.xsd"＞

	<beans profile="dev">
		<bean id="myBean" class="com.demo.MyBean" />
	</beans>

	<beans profile="prod">
		<bean id="myBean" class="com.demo.MyBean" />
	</beans>
</beans>
```

Spring 在确定激活的 profile 时，依赖两个独立的属性：`spring.profiles.active` 和 `spring.profiles.default`。

有多种方式来设置：

- 作为 `DispatcherServlet` 的初始化参数
- 作为 Web 应用的上下文参数
- 作为 JNDI 条目
- 作为环境变量
- 作为 JVM 的系统属性
- 在集成测试类上，使用 `@ActiveProfiles` 注解

```xml
<!-- 为上下文设置默认的 profile -->
<context-param>
	<param-name>spring.profiles.default</param-name>
	<param-value>dev</param-value>
	<!-- 也可以设置多个 profile -->
	<!-- <param-value>dev1,dev2</param-value> -->
</context-param>
...
<!-- 为 Servlet 设置默认的 profile -->
<servlet>
	<servlet-name>dispatcherServlet</servlet-name>
	<servlet-class>
		org.springframework.web.servlet.DispatcherServlet
	</servlet-class>
	<init-param>
		<param-name>spring.profiles.default</param-name>
		<param-value>dev</param-value>
	</init-param>
</servlet>
```

## 条件化的 bean

```java
@Bean
@Conditional(MagicExistsCondition.class)
public MagicBean magicBean() {
	return new MagicBean();
}
```

```java
public class MagicExistsCondition implements Condition {
	@Override
	public boolean matches(
		ConditionContext context,
		AnnotatedTypedMetadata metadata) {
		Environment env = context.getEnvironment();
		return env.containsProperty("magic");
	}
}
```

其中的 `ConditionContext`：

- 借助 `getRegistry()` 返回的 `BeanDefinitionRegistry` 检查 bean 定义；
- 借助 `getBeanFactory()` 返回的 `ConfigurableListableBeanFactory` 检查 bean 是否存在，甚至探查 bean 的属性；
- 借助 `getEnvironment()` 返回的 `Environment` 检查环境变量是否存在以及值；
- 读取并探查 `getResourceLoader()` 返回的 `ResourceLoader` 所加载的资源；
- 借助 `getClassLoader()` 返回的 `ClassLoader` 加载并检查类是否存在。

其中的 `AnnotatedTypeMetadata` 能检查 `@Bean` 注解的方法上还有什么其他的注解以及它们的属性。

## 处理自动装配的歧义性

### 标示首选的 bean

```java
@Component
@Primary
public class IceCream implements Dessert {/*bla*/}

@Bean
@Primary
public Dessert iceCream() {/*bla*/}
```

```xml
<bean id="iceCream" class="com.demo.IceCream" primary="true" />
```

### 限定自动装配的 bean

```java
@Autowired
@Qualifier("iceCream") //指定 bean 的限定符
public void setDessert(Dessert dessert) {
	this.dessert = dessert
}
```

bean 的限定符和 ID 本质上是独立的。bean 创建时若不指定限定符，则默认用 ID 作为限定符。

若使用默认限定符的话，当重命名类名之后，之前使用 `@Qualifier` 注入的地方就会报错了。可以在 bean 声明上也添加 `@Qualifier`。

```java
@Component
@Qualifier("cold")
public class IceCream implements Dessert {}
```

有时需要用多个限定符来确定唯一的 bean，但是又不能用多个 `@Qualifier` （Java 8 开始允许出现重复的注解，只要该注解本身带有 `@Repeatable` 注解，但是 `@Qualifier` 没有 `@Repeatable` 注解）。此时可以使用 `@Qualifier` 来实现自定义的注解。

```java
@Target({ElementType.CONSTRUCTOR, ElementType.FIELD,
         ElementType.METHOD, ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Qualifier
public @interface Creamy {}
```

这样就可以用多个自定义的注解来确定唯一的 bean。

## bean 的作用域

- 单例（Singleton）：整个应用中，只创建 bean 的一个实例。（默认）
- 原型（Prototype）：每次注入或通过 Spring 上下文获取时都会创建一个新的 bean 实例。
- 会话（Session）：在 Web 应用中，为每个会话创建一个 bean 实例。
- 请求（Request）：在 Web 应用中，为每个请求创建一个 bean 实例。

```java
@Component
@Scope(ConfigurableBeanFactory.SCOPE_PROTOTYPE)
public class Notepad {}

@Bean
@Scope(ConfigurableBeanFactory.SCOPE_PROTOTYPE)
public Notepad notepad() {}
```

```xml
<bean id="notepad" class="com.myapp.Notepad" scope="prototype" />
```

### 使用会话和请求作用域

```java
@Component
@Scope(value=WebApplicationContext.SCOPE_SESSION,
       proxyMode=ScopedProxyMode.INTERFACES)
public ShoppingCart cart() {...}
```

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xmlns:aop="http://www.springframework.org/schema/aop"
  xsi:schemaLocation="
    http://www.springframework.org/schema/aop
    http://www.springframework.org/schema/aop/spring-aop.xsd
    http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="cart" class="com.demo.ShoppingCart" scope="session">
    	<!-- 默认使用 CGLib 创建目标类的代理 -->
    	<aop:scoped-proxy />
		<!-- 设置成基于接口的代理 -->
    	<aop:scoped-proxy proxy-target-class="false" />
    </bean>
</beans>
```

由于会话和请求作用域的 bean 是跟随着会话和请求而创建的，因此在初始化 Spring 上下文的时候没法注入到其他 bean 里去。

通过 `proxyMode` 属性来解决这个问题。{@link ScopedProxyMode}
Spring 会将代理类注入到其他 bean 中。在运行期，代理类会负责将方法调用转发给作用域内真正的 bean。

- INTERFACES：创建一个 JDK 动态代理，该代理实现了目标接口；
- TARGET_CLASS：使用 CGLib 创建目标类的代理；
- NO：不创建作用域代理（对于作用域 bean 没用）
- DEFAULT：通常默认值为 `NO`，也可以在 component-scan 层进行自定义配置。

## 运行时值注入

Spring 提供了两种运行时求值的方式：

- 属性占位符（Property placeholder）
- Spring 表达式语言（SpEL）

### 注入外部的值

为了使用占位符，需要配置一个 `PropertyPlaceholderConfigurer` bean 或 `PropertySourcesPlaceholderConfigurer` bean。

从 Spring 3.1 开始，推荐 `PropertySourcesPlaceholderConfigurer`，它能基于 `Environment` 及其属性源来解析占位符。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xmlns:context="http://www.springframework.org/schema/context"
  xsi:schemaLocation="
  	http://www.springframework.org/schema/beans
  	http://www.springframework.org/schema/beans/spring-beans.xsd
  	http://www.springframework.org/schema/context
  	http://www.springframework.org/schema/context/spring-context.xsd">

	<context:property-placeholder
      ignore-resource-not-found="false"
      location="classpath:/com/soundsystem/app.properties" />

</beans>
```

```java
@Configuration
@PropertySource("classpath:/com/soundsystem/app.properties")
public class ExpressiveConfig {
	@Autowired
	Environment env;

  @Bean
  public static PropertySourcesPlaceholderConfigurer placeHolderConfigurer() {
  	return new PropertySourcesPlaceholderConfigurer();
  }

	@Bean
	public BlankDisc disc() {
		return new BlankDisc(
			env.getProperty("disc.title"),
			env.getProperty("disc.artist"));
	}
}
```

依赖自动装配时

```java
public BlankDisc(
	@Value("${disc.title}") String title,
	@Value("${disc.artist}") String artist) {
	//blabla
}
```

`getProperty` 有 4 个重载方法：

- `String getPropety(String key)`
- `String getProperty(String key, String defaultValue)`
- `T getProperty(String key, Class<T> type)`
- `T getProperty(String key, Class<T> type, T defaultValue)`

`Environment` 的其他方法：

- `getRequiredProperty()` 获取到 null 会抛出 `IllegalStateException`
- `containsProperty()`
- `getPropertyAsClass()`
- `String[] getActiveProfiles()`
- `String[] getDefaultProfiles()`
- `boolean acceptsProfiles(String... profiles)`

**关于 `@PropertySource` 的解析**

`org.springframework.context.annotation.ConfigurationClassParser#doProcessConfigurationClass`

### 使用 Spring 表达式进行装配

SpEL 的特性包括：

- 使用 bean 的 ID 来引用 bean；
- 调用方法和访问对象的属性；
- 对值进行算术、关系和逻辑运算；
- 正则表达式匹配；
- 集合操作。

**表示字面量**

`#{3.14}` `#{1.23E4}` 浮点值
`#{'Hello'}` 字符串
`#{false}` 布尔值

**引用 bean、属性和方法**

`#{sgtPeppers}` 引用一个 bean
`#{sgtPeppers.fieldA}` bean 的属性
`#{artistSelector.selectArtist()}` 调用 bean 的方法
`#{artistSelector.selectArtist()?.toUpperCase()}` 调用返回值的方法。`?` 会进行判空，为空则直接返回 null，因而不会空指针异常

**在表达式中使用类型**

`#{T(java.lang.Math).PI}` 访问类的静态成员
`#{T(java.lang.Math).random()}` 访问类的静态方法

**SpEL 运算符**

|运算符类型|运算符                              |
|:---------|:-----------------------------------|
|算术运算  |+、-、\*、/、%、^                   |
|比较运算  |<、>、==、<=、>=、lt、gt、eq、le、ge|
|逻辑运算  |and、or、not、\|                    |
|条件运算  |?: (ternary)、?: (Elvis)            |
|正则表达式|matches                             |

`#{scoreboard.score > 1000 ? "Winner" : "Loser"}` 三元运算
`#{disc.title ?: 'Default Title'}` 为 null 的话取默认值；`?:` 也可以简写成 `:`

**计算正则表达式**

`#{user.email matches '[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\\.com}`

**计算集合**

`#{jukebox.songs[4].title}` 引用第 5 个元素的属性（索引从 0 开始）
`#{jukebox.songs.?[artist eq 'Aerosmith']}` 对集合进行过滤。除此之外，`.^[]` 和 `.$[]` 分别查询第一个和最后一个匹配项；`.![]` 取每个元素的某个字段，并组成一个新的集合

# 第 4 章 面向切面的 Spring

### 定义 AOP 术语

**通知（Advice）**

切面的工作被称为通知。通知定义了“何时”、“何事”。

Spring 切面可以应用 5 种类型的通知：

- 前置通知（Before）：在目标方法调用之前
- 后置通知（After）：在目标方法完成之后，此时不关心方法的输出是什么
- 返回通知（After-returning）：在目标方法成功执行之后
- 异常通知（After-throwing）：在目标方法抛出异常后
- 环绕通知（Around）：在目标方法调用之前和之后

**连接点（Join point）**

连接点是在应用执行过程中能够插入切面的一个点。比如调用方法时、抛出异常时等等。

**切点（Pointcut）**

切点定义了“何处”。

**切面（Aspect）**

切面是通知和切点的结合。

**引入（Introduction）**

引入允许我们向现有的类添加新方法或属性。

**织入（Weaving）**

织入是把切面应用到目标对象并创建新的代理对象的过程。在目标对象的生命周期里有多个点可以进行织入：

- 编译期：这种方式需要特殊的编译器。AspectJ 的织入编译器就是以这种方式织入切面的。
- 类加载期：切面在目标类加载到 JVM 时被织入。这种方式需要特殊的类加载器（ClassLoader），它可以在目标类被引入应用之前增强该目标类的字节码。AspectJ 5 的加载时织入（load-time weaving,LTW）就支持以这种方式织入切面。
- 运行期：切面在应用运行的某个时刻被织入，一般在织入切面时，AOP 容器会为目标对象动态地创建一个代理对象。Spring AOP 就是以这种方式织入切面的。

### Spring 对 AOP 的支持

Spring 提供了 4 种类型的 AOP 支持：

- 基于代理的经典 Spring AOP；
- 纯 POJO 切面；
- `@AspectJ` 注解驱动的切面；
- 注入式 AspectJ 切面（适用于 Spring 各版本）

## 通过切点来选择连接点

|AspectJ 指示器|描述                                                                                       |
|:----------|:---------------------------------------------------------------------------------------------|
|arg()      |限制连接点匹配参数为指定类型的执行方法                                                        |
|@args()    |限制连接点匹配参数由指定注解标注的执行方法                                                    |
|execution()|用于匹配是连接点的执行方法                                                                    |
|this()     |限制连接点匹配 AOP 代理的 bean 引用为指定类型的类                                             |
|target     |限制连接点匹配目标对象为指定类型的类                                                          |
|@target()  |限制连接点匹配特定的执行对象，这些对象对应的类要具有特定类型的注解                            |
|within()   |限制连接点匹配指定的类型                                                                      |
|@within()  |限制连接点匹配指定注解所标注的类型（当使用 Spring AOP 时，方法定义在由指定的注解所标注的类里）|
|@annotation|限定匹配带有指定注解的连接点                                                                  |

### 编写切点

```
 execution(* concert.Performance.perform(..)) && within(concert.*)
|----1----|2|---------3---------|---4---|-5-|-6-|--------7-------|
```

1. 在方法执行时触发
2. 返回任意类型
3. 方法所属的类
4. 方法名
5. 使用任意参数
6. 通过操作符连接多个指示器。操作符包括 `&&`、`||`、`!`，也可以用 `and`、`or`、`not` 代替

### 在切点中选择 bean

Spring 还引入了一个新的 `bean()` 指示器，用来通过 bean ID 来标识 bean。

```
execution(* concert.Performance.perform(..)) and bean('woodstock')
```

## 使用注解创建切面

### 定义切面

|注解           |通知                                    |
|:--------------|:---------------------------------------|
|@After         |通知方法会在目标方法返回或抛出异常后调用|
|@AfterReturning|通知方法会在目标方法返回后调用          |
|@AfterThrowing |通知方法会在目标方法抛出异常后调用      |
|@Around        |通知方法会将目标方法封装起来            |
|@Before        |通知方法会在目标方法调用之前执行        |


```java
//表明这个类是一个切面，但不表明会启用切面功能。
//而且依然是用 Spring 基于代理的切面，而不是 AspectJ 的切面。
@Aspect
public class Audience {
	/**
	 * 这个方法的内容不重要，只是为了给 @Pointcut 提供一个附着点。
	 * 方法名会作为切点的标识符。这样就可以重用同一个切点。
	 */
	@Pointcut("execution(** concert.Performance.perform(..))")
	public void performance() {}

	/**
	 * args() 中的参数名要与切点方法中的参数名相匹配
	 */
	@Pointcut(
		"execution(** concert.Performance.performAfterEncore(String)"
		+ "and args(encoreContent)")
	public void encore(String encoreContent) {
		System.out.println("Wow!" + encoreContent);
	}

	@Before("performance()")
	public void silenceCellPhones() {
		//blabla
	}

	@AfterReturning("performance()")
	public void applause() {
		//blabla
	}

	@Around("performance()")
	public void watchPerformance(ProceedingJoinPoint jp) {
		try {
			//blabla
			jp.proceed();// 需要手动调用切点方法
			//blabla
		} catch(Throwable e) {
			//blabla
		}
	}
}
```

**启用切面**

```java
@Configuration
@EnableAspectJAutoProxy //启用 AspectJ 自动代理
@ComponentScan
public class ConcertConfig {
	@Bean
	public Audience audience() {
	return new Audience();
	}
}
```

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xmlns:context="http://www.springframework.org/schema/aop"
  xmlns:aop="http://www.springframework.org/schema/aop"
  xsi:schemaLocation="http://www.springframework.org/schema/aop
    http://www.springframework.org/schema/aop/spring-aop.xsd
    http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans.xsd
    http://www.springframework.org/schema/context
    http://www.springframework.org/schema/context/spring-context.xsd">

    <context:component-scan base-package="concert" />
	<!-- 启用 AspectJ 自动代理 -->
    <aop:aspectj-autoproxy />
    <bean class="concert.Audience" />
</beans>
```

### 通过注解引入新功能

```java
@Aspect
public class EncoreableIntroducer {
	@DeclareParents(value="concert.Performance+", defaultImpl=DefaultEncoreable.class)
	public static Encoreable encoreable;
}
```

`@DeclareParents` 注解由三部分组成：

- `value` 属性指定了哪种类型的 bean 要引入该接口。加号表示是它的所有子类型而不是它本身。
- `defaultImpl` 属性指定了为引入功能提供实现的类。
- `@DeclareParents` 所标注的静态属性指明了要引入的接口。

此外要将切面声明为 bean：

```xml
<bean class="concert.EncoreableIntroducer" />
```

## 在 XML 中声明切面

|AOP 配置元素            |用途                                                           |
|:-----------------------|:--------------------------------------------------------------|
|`<aop:advisor>`          |定义 AOP 通知器                                                |
|`<aop:after>`            |定义 AOP 后置通知（不管被通知方法是否执行成功）                |
|`<aop:after-returning>`  |定义 AOP 返回通知                                              |
|`<aop:after-throwing>`   |定义 AOP 异常通知                                              |
|`<aop:around>`           |定义 AOP 环绕通知                                              |
|`<aop:aspect>`           |定义一个切面                                                   |
|`<aop:aspectj-autoproxy>`|启用 `@AspectJ` 注解驱动的切面                                  |
|`<aop:before>`           |定义一个 AOP 前置通知                                          |
|`<aop:config>`           |顶层的 AOP 配置元素。大多数的 `<aop:*>` 元素必须包含在此元素内。|
|`<aop:declare-parents>`  |以透明的方式为被通知的对象引入额外的接口                       |
|`<aop:pointcut>`         |定义一个切点                                                   |

```xml
<aop:config>
	<aop:aspect ref="audience">
		<aop:pointcut
		  id="performance"
		  expression="execution(** concert.Performance.perform(..))" />
		<aop:before
		  pointcut-ref="performance"
		  method="takeSeats" />
	</aop:aspect>

	<!-- 引入 -->
	<!-- default-impl 也可以换成 delegate-ref="{beanId}" -->
	<aop:aspect>
		<aop:declare-parents
		  types-matching="concert.Performance+"
		  implement-interface="concert.Encoreable"
		  default-impl="concert.DefaultEncoreable" />
	</aop:aspect>
</aop:config>
```

## 切面的一些实践细节

### 连接点对象 ProceedingJoinPoint

假设被切面包裹的类如下

```java
package com.example;

pubilc class ExampleTarget {
    public void test() {
        // blabla
    }
}
```



`jp.getKind()`

method-execution（应该还有其他枚举值）

`jp.getSignature().getDeclaringTypeName()`

com.example.ExampleTarget

`jp.getSignature().getName()`

test

`jp.getTarget().toString()`

com.example.ExampleTarget@15a3d5a

后面的十六进制代码是 Object 类默认的基于对象内存地址计算出的 hashCode。

`jp.getTarget().getClass().toString()` `jp.getSignature().getDeclaringType().toString()`

class com.example.ExampleTarget

# 事务管理

## 事务管理器

Spring 的事务管理器都是 `PlatformTransactionManager` 的实现。

### JDBC 事务

```xml
<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
    <property name="dataSource" ref="dataSource" />
</bean>
```

事务管理器通过调用 `java.sql.connection` 来管理事务。

### Hibernate 事务

```xml
<bean id="transactionManager" class="org.springframework.orm.hibernate3.HibernateTransactionManager">
    <property name="sessionFactory" ref="sessionFactory" />
</bean>
```

事务管理器通过调用 `org.hibernate.Transaction` 对象来管理事务。

### JPA 事务

```xml
<bean id="transactionManager" class="org.springframework.orm.jpa.JpaTransactionManager">
    <property name="entityManagerFactory" ref="entityManagerFactory" />
</bean>
```

### JTA 事务

```xml
<bean id="transactionManager" class="org.springframework.transaction.jta.JtaTransactionManager">
    <property name="transactionManagerName" value="java:/TransactionManager" />
</bean>
```

## 在 Spring 中的编码事务

调用 `TransactionTemplate.execute()` 方法。

```xml
<bean id="txTemplate" class="org.springframework.transaction.support.TransactionTemplate">
    <property name="transactionManager" ref="transactionManager" />
</bean>
```

## 声明式事务

### 定义事务属性

事务属性描述了事务策略，包括 5 个方面。

**传播行为**

`PROPAGATION_MANDATORY`

表示该方法必须在事务中运行。如果当前事务不存在，则会抛出异常。

`PROPAGATION_NESTED`

表示如果当前已经存在一个事务，那么该方法将会在嵌套事务中运行。嵌套的事务可以独立于当前事务进行单独地提交或回滚。如果当前事务不存在，则其行为与 `PROPAGATION_REQUIRED` 一样。各厂商对这种行为的支持有所差异。

`PROPAGATION_NEVER`

表示当前方法不应该运行在事务上下文中。如果当前正有一个事务在运行，则会抛出异常。

`PROPAGATION_NOT_SUPPORTED`

表示该方法不应该运行在事务中。如果存在当前事务，在该方法运行期间，当前事务将被挂起。如果使用 `JTATransactionManager` 的话，则需要访问 `TransactionManager`。

`PROPAGATION_REQUIRED`

表示当前方法必须运行在事务中。如果不存在当前事务，则会启动一个新事务。

`PROPAGATION_REQUIRES_NEW`

表示当前方法必须运行在它自己的事务中。如果不存在当前事务，则会启动一个新事务。如果存在当前事务，在该方法执行期间，当前事务会被挂起。如果使用 `JTATransactionManager` 的话，则需要访问 `TransactionManager`。 

`PROPAGATION_SUPPORTS`

表示当前方法不需要事务上下文，但是如果存在当前事务的话，那么该方法会在这个事务中运行。

**隔离级别**

并发事务可能产生的问题：

- 脏读：A 读取了 B 改写的但未提交的数据，然后 B 回滚了，因此 A 读取的数据是无效的。
- 不可重复读：A 先后执行两次相同的查询，但两次数据不同，因为 B 在之间修改了数据。
- 幻读：A 读取了几行数据，然后 B 插入了一些数据，接着 A 在查询时发现多了一些原本不存在的记录。

不同的隔离级别：

`ISOLATION_DEFAULT` 使用后端数据库默认的隔离级别。

`ISOLATION_READ_UNCOMMITTED` 允许读取尚未提交的数据变更。可能会导致脏读、幻读和不可重复读。

`ISOLATION_READ_COMMITTED` 允许读取并发事务已经提交的数据。可以阻止脏读，但是幻读或不可重复读仍有可能发生。

`ISOLATION_REPEATABLE_READ` 对同一字段的多次读取结果是一致的。除非数据是被本事务自己所修改。可以阻止脏读和不可重复读，但幻读仍有可能。

`ISOLATION_SERIALIZABLE` 完全服从 ACID 的隔离级别，确保阻止脏读、不可重复读以及幻读。这是最慢的事务隔离级别，因为它通常是通过完全锁定事务相关的数据库表来实现的。

**只读**

若事务只对后端的数据库进行读操作，数据库可以利用事务的只读特性来进行一些特定的优化。因为只读优化实在事务启动时由数据库实施的，所以只对那些具备启动新事务的传播行为有效。

**事务超时**

因为超时时钟在事务创建时启动，所以只对那些具备启动新事务的传播行为有效。

**回滚规则**

默认情况下，事务只有在遇到运行期异常时才会回滚，遇到检查型异常时不会。

### 在 XML 中声明事务

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xmlns:aop="http://www.springframework.org/schema/aop"
  xmlns:tx="http://www.springframework.org/schema/tx"
  xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans-3.0.xsd
    http://www.springframework.org/schema/aop
    http://www.springframework.org/schema/aop/spring-aop-3.0.xsd
    http://www.springframework.org/schema/tx
    http://www.springframework.org/schema/tx/spring-tx-3.0.xsd">

    <!-- 启动注解驱动的事务管理 @Transactional -->
    <!-- 默认为 id="transactionManager" 的 bean，若不是这个 id 则需写明 -->
    <tx:annotation-driven transaction-manager="txManager" />

    <!-- transaction-manager 默认为 id="transactionManager" 的 bean，若不是这个 id 则需写明 -->
    <tx:advice id="txAdvice" transaction-manager="txManager">
        <tx:attributes>
            <tx:method name="save*" propagation="REQUIRED" />
            <tx:method name="*" propagation="SUPPORTS" read-only="true" />
        </tx:attributes>
    </tx:advice>

</beans>
```

5 个事务属性对应的 xml 属性

|属性       |含义                            |
|:----------|:-------------------------------|
|isolation  |指定事务的隔离级别              |
|propagation|定义事务的传播规则              |
|read-only  |指定事务为只读                  |
|rollback-for, no-rollback-for|rollback-for 指定事务对于哪些检查型异常应当回滚而不提交；no-rollback-for 指定事务对于哪些异常应当继续执行而不回滚。|
|timeout    |对于长时间运行的事务定义超时时间|
# 第 5 章 构建 Spring Web 应用程序

# 第 6 章 渲染 Web 视图

# 第 7 章 Spring MVC 的高级技术

## 概述

1. Web 应用服务器收到 HTTP 请求，根据 web.xml 中的配置，交给 DispatcherServlet 处理；
2. DispatcherServlet 根据请求中的信息以及配置的 HandlerMapping 搜索具体的处理器 handler。
3. 用 HandlerAdapter 对 Handler 进行封装。DispatcherServlet 调用 HandlerAdapter.handle()，而 adapter 根据 handler 的不同采用不同的方法调用具体的方法；
4. 处理器返回一个 ModelAndView 给 DispatcherServlet，其中包含了视图/视图名以及数据模型；
5. 根据视图名解析得到实际的 View 对象（若上一步得到的是视图名而不是视图对象）；
6. 用数据模型对视图对象进行渲染；
7. 向客户端返回渲染结果。

## IOC

### 资源访问

Spring 的 Resource 接口用于访问各类资源。

继承关系

```
Resource
|- WritableResource
\- | AbstractResource
   | |- ByteArrayResource
   | |- InputStreamResource
   | |- ClassPathResource
   | |- PortletContextResource
   | |- ServletContextResource
   | |- DescriptiveResource
   | |- UrlResource
   +-+- PathResource
   \-+- FileSystemResource
```

可使用 PathMatchingResourcePatternResolver 通过资源前缀和 Ant 风格的通配符来自动加载 Resource。

资源地址前缀：

- `classpath:` 类路径。
    - `classpath:` 和 `classpath:/` 等价。
    - `classpath:` 加载第一个匹配的资源，`classpath*:` 加载所有匹配的资源。
- `file:` 使用 UrlResource 从文件系统目录中加载，可使用绝对或相对路径。
- `http://` 使用 UrlResource 从 Web 服务器中装载资源。
- `ftp://` 使用 UrlResource 从 FTP 服务器中装载资源。
- `没前缀` 根据 ApplicationContext 的具体实现类采用对应类型的 Resource。

Ant 风格的资源地址：

- `?` 匹配文件名中的一个字符。
- `*` 匹配文件名中的任意字符。
- `**` 匹配多层路径。

项目部署后没法通过 Resource#getFile() 操作文件，要使用 Resource#getInputStream()。

## DispatcherServlet

### 配置

DispatcherServlet 可通过 `<servlet>` 的 `<init-param>` 配置参数：

```xml
<servlet>
    <servlet-name>myServlet</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <init-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>/WEB-INF/spring/webApplicationContext.xml</param-value>
    </init-param>
    <load-on-startup>1</load-on-startup>
</servlet>
```

- namespace：DispatcherServlet 的命名空间，默认为 `{servlet-name}-servlet`，用以构造 servlet 配置文件的路径。完整路径是“WEB-INF/{namespace}.xml”；
- contextConfigLocation：指定 servlet 配置文件的路径，可为多个，逗号分隔；
- publishContext：boolean 类型，默认为 true。DispatcherServlet 根据此属性决定是否将 WebApplicationContext 发布到 ServletContext 属性列表中，以便可以从 ServletContext 找到 WebApplicationContext 实例，对应的属性名为 DispatcherServlet#getServletContextAttributeName() 返回的值；
- publishEvents：boolean 类型，默认为 true。当 DispatcherServlet 处理完一个请求后，是否需要向容器发布一个 ServletRequestHandledEvent 事件。如果容器中没有任何时间监听器，可设为 false，以提高性能。

### 内部逻辑

```java
protected void initStrategies(ApplicationContext context) {
    initMultipartResolver(context);//多部分请求解析器
    initLocaleResolver(context);//本地化解析器
    initThemeResolver(context);//主题解析器
    initHandlerMappings(context);//处理器映射器
    initHandlerAdapters(context);//处理器适配器
    initHandlerExceptionResolvers(context);//处理器异常解析器
    initRequestToViewNameTranslator(context);//请求到视图名翻译器
    initViewResolvers(context);//视图解析器
    initFlashMapManager(context);//FlashMap 管理器，用于在不同请求间共享数据
｝
```

org/springframework/web/servlet/DispatcherServlet.properties 中指定了默认组件。

这些组件都实现了 `org.springframework.core.Ordered` 接口，order 值越小优先级越高。

#### 消息转换器 HttpMessageConverter

DispatcherServlet 默认安装了 AnnotationMethodHandlerAdapter 作为 HandlerAdapter 的实现。该实现使用 HttpMessageConverter 将请求信息转换为对象。

- `Boolean canRead(Class<?> clazz, MediaType mediaType)`：指定转换器可以将请求信息转换为 clazz 类型对象，同时指定支持的 MIME 类型；
- `Boolean canWrite(Class<?> clazz, MedaiType mediaType)`：指定转换器可以将 clazz 类型对象写入响应流，同时指定响应流支持的 MIME 类型；
- `List<MediaType> getSupportedMediaTypes()`：转换器支持的媒体类型；
- `T read(Class<? extends T> clazz, HttpInputMessage inputMessage)`：将请求信息转换为对象；
- `void write(T t, MediaType contentType, HttpOutputMessage outputMessage)`：将对象写到响应流，同时指定响应的媒体类型。

HttpMessageConverter 的实现类：

- StringHttpMessageConverter：将请求信息转换为字符串；
    - 可读取所有媒体类型（*/*），可通过设置 `supportedMediaTypes 属性指定媒体类型；
    - 响应的媒体类型为 text/plain；
- FormHttpMessageConverter：将表单数据读取到 MutiValueMap 中；
    - 可读取 application/x-www-form-urlencoded 的媒体类型，但不支持 multipart/form-data 类型；
    - 可写 application/x-www-form-urlencoded 和 multipart/form-data 类型的响应；
- XmlAwareFormHttpMessageConverter：扩展自 FormHttpMessageConverter，如果部分表单数据是 xml，可用此转换器读取；
- ResourceHttpMessageConverter：读写 `org.springframework.core.io.Resource` 对象；
    - 可读所有媒体类型；
    - 若类路径下提供了 JAF（Java Activation Framework），则根据 Resource 的类型指定响应的媒体类型，否则响应的类型为 application/octet-stream；
- BufferedImageHttpMessageConverter：读写 `BufferedImage` 对象；
    - 可读所有媒体类型；
    - 返回 `BufferedImage` 相应的媒体类型，也可通过 contentType 显式指定；
- ByteArrayHttpMessageConverter：读写二进制数据；
    - 可读所有媒体类型，可通过 `supportedMediaTypes` 属性指定媒体类型；
    - 响应类型为 application/octet-stream；
- SourceHttpMessageConverter：读写 `javax.xml.transform.Source` 类型的数据；
    - 可读 text/xml 和 application/xml 类型；
    - 响应类型为 text/xml 或 application/xml；
- MarshallingHttpMessageConverter：通过 Spring 的 `org.springframework.oxm.Marshaller` 和 `Unmarshaller` 读写 xml 消息；
    - 可读 text/xml 和 application/xml 类型；
    - 响应类型为 text/xml 或 application/xml；
- Jaxb2RootElementHttpMessageConverter：通过 JAXB2 读写 xml 消息，将请求信息转换到注解 `@XmlRootElement` 或 `@XmlType` 的类中；
    - 可读 text/xml 和 application/xml 类型；
    - 响应类型为 text/xml 或 application/xml；
- MappingJacksonHttpMessageConverter：利用 Jackson 开源类包的 `ObjectWrapper` 读写 JSON 数据；
    - 可读 application/json 类型；
    - 响应类型为 application/json；
- RssChannelHttpMessageConverter：能读写 RSS 种子消息；
    - 可读 application/rss+xml 类型，转换为 `com.sun.syndication.feed.rss.Channel` 类型；
    - 响应类型为 application/rss+xml；
- AtomFeedHttpMessageConverter：和 RssChannelHttpMessageConverter 能够读写 RSS 种子消息；
    - 可读 application/atom+xml 类型，转换为 `com.sun.syndication.feed.atom.Feed` 类型；
    - 响应类型为 application/atom+xml。

AnnotationMethodHandlerAdapter 默认装配了以下转换器：

- ByteArrayHttpMessageConverter
- StringHttpMessageConverter
- SourceHttpMessageConverter
- org.springframework.http.converter.xml.XmlAwareFormHttpMessageConverter


# 第 8 章 使用 Spring Web Flow

# 第 9 章 保护 Web 应用

## Spring Security 的模块

|模块                    |描述                                                                            |
|:-----------------------|:-------------------------------------------------------------------------------|
|ACL                     |支持通过访问控制列表（access control list,ACL）为域对象提供安全性               |
|切面（Aspects）         |一个很小的模块，当使用 Spring Security 注解时，会使用基于 AspectJ 的切面，而不是使用标准的 Spring AOP|
|CAS 客户端（CAS Client）|提供与 Jasig 的中心认证服务（Central Authentication Service, CAS）进行集成的功能|
|配置（Configuration）   |包含通过 XML 和 Java 配置 Spring Security 的功能支持                            |
|核心（core）            |提供 Spring Security 基本库                                                     |
|加密（Cryptography）    |提供了加密和密码编码功能                                                        |
|LDAP                    |支持基于 LDAP 进行认证                                                          |
|OpenID                  |支持使用 OpenID 进行集中式认证                                                  |
|Remoting                |提供了对 Spring Remoting 的支持                                                 |
|标签库（Tag Library）   |Spring Security 的 JSP 标签库                                                   |
|Web                     |提供了 Spring Security 基于 Filter 的 Web 安全性支持                            |
# 第 10 章 通过 Spring 和 JDBC 征服数据库

# 第 11 章 使用对象-关系映射持久化数据

## JPA

基于 JPA 的应用需要使用 `EntityManagerFactory` 的实现类来获取 `EntityManager` 实例。JPA 定义了两种类型的实体管理器：

- 应用程序管理类型：程序负责打开和关闭实体管理器并在事务中进行控制。
- 容器管理类型：Java EE 负责创建和管理实体管理器，程序通过注入或 JNDI 来获取。

分别对应的 Spring 工厂 bean：

- `LocalEntityManagerFactoryBean`
- `LocalContainerEntityManagerFactoryBean`

对于基于 Spring 的应用程序来说两者的区别只在于在 Spring 上下文中如何配置。

### 配置应用程序管理类型的 JPA

首先定义一个持久化单元文件 persistence.xml，然后用它作为参数创建 `LocalEntityManagerFactoryBean`。

### 配置容器管理类型的 JPA

`jpaVendorAdapter` 属性指定 JPA 实现。

- `EclipseLinkJpaVendorAdapter`
- `HibernateJpaVendorAdapter`
- `OpenJpaVendorAdapter`
- `TopLinkJpaVendorAdapter` *Spring 3.1 中已废弃*

```java
@Bean
public LocalContainerEntityManagerFactoryBean entityManagerFactory(DataSource dataSource, JpaVendorAdapter jpaVendorAdapter) {
    LocalContainerEntityManagerFactoryBean emfb =
        new LocalContainerEntityManagerFactoryBean();
    emfb.setDataSource(dataSource);
    emfb.setJpaVendorAdapter(jpaVendorAdapter);
    emfb.setPackagesToScan("com.example");
    return emfb;
}
```

### Spring-data JPA

启动 Spring Data JPA 自动扩展 `Repository` 接口的功能。

```xml
<jpa:repositories base-package="com.example"
    transaction-manager-ref="transactionManager"
    entity-manager-factory-ref="entityManagerFactory" />
```

# 第 12 章 使用 NoSQL 数据库

# 第 13 章 缓存数据

# 第 14 章 保护方法应用

# 第 15 章 使用远程服务

# 第 16 章 使用 Spring MVC 创建 REST API

# 第 17 章 Spring 消息

## 使用 JMS 发送消息

### 在 Spring 中搭建消息代理

**创建连接工厂**

1. 创建连接工厂。`<bean id="connectionFactory" class="org.apache.activemq.spring.ActiveMQConnectionFactory"/>
2. 默认假设 ActiveMQ 代理监听 61616 端口。也可以指定代理的 URL。`<bean id="connectionFactory" class="org.apache.activemq.spring.ActiveMQConnectionFactory" p:brokerURL="tcp://localhost:61616"/>

也可以使用 ActiveMQ 的命名空间来配置。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       ...
       xmlns:amq="http://activemq.apache.org/schema/core"
       xsi:shemaLocation:"http://activemq.apache.org/schema/core
                   http://activemq.apache.org/schema/core/activemq-core.xsd">
	<amq:connectionFactory id="connectionFactory" brokerURL="tcp://localhost:61616"/>
</beans>
```

**声明消息目的地**

使用特定消息代理实现类在 Spring 中声明一个 bean。

```xml
<!-- 队列 -->
<bean id="queue" class="org.apache.activemq.command.ActiveMQQueue" c:_="{queueName}"/>
<!-- 主题 -->
<bean id="topic" class="org.apache.activemq.command.ActiveMQTopic" c:_="{topicName}"/>
```

也可以使用 ActiveMQ 的命名空间来配置。

```xml
<amq:queue id="queue" physicalName="{queueName}"/>
<amq:topic id="topic" physicalName="{topicNmae}"/>
```

**使用 JMS 模板**

将 JmsTemplate 声明为 bean。其中默认的目的地配置是可选的。也可以把目的地对象装配进来：`p:defaultDestination-ref="{destBean}"`。

```xml
<bean id="jmsTemplate"
      class="org.springframework.jms.core.JmsTemplate"
      c:_="connectionFactory"
      p:defaultDestinationName="{destName}"/>
```

生产者代码

```java
public class Producer {
    @Autowired
    private JmsOperations jmsOperations; // JmsTemplate 实现了此接口
    /**
     * 发送的时候定义消息的创建逻辑
     */
    public void sendMessage(MyObj obj) {
        jmsOperations.send("dest.name",
            new MessageCreator() {
                public Message createMessage(Session session) throws JMSException {
                	return session.createObjectMessage(obj);
                }               
            });
    }
    
    /**
     * 发送的时候使用转换器
     */
    public void sendMessageWithAutoConversion(MyObj obj) {
        jmsOperations.convertAndSend(obj);
    }
}
```

可以使用 Spring 预置的转换器，也可以实现 MessageConverter 接口自定义一个转换器。

- MappingJacksonMessageConverter：使用 Jackson JSON 库实现消息与 JSON 格式之间的相互转换。
- MappingJackson2MessageConverter：使用 Jackson2 JSON 库实现消息与 JSON 格式之间的相互转换。
- MarshallingMessageConverter：使用 JAXB 库实现消息与 XML 格式之间的转换。
- SimpleMessageConverter：实现 String 与 TextMessage，字节数组与 ByteMessage，Map 与 MapMessage，Serializable 与 ObjectMessage 之间的转换。

消费者代码

```java
public MyObj receiveMessage() {
    try {
        // receive() 会以阻塞的方式收取消息
        ObjectMessage message = (ObjectMessage) jmsOperations.receive();
        return (MyObj) message.getObject();
    } catch (JMSException e) {
        // ...
    }
}

public MyObj receiveMessageWithAutoConversion() {
    return (MyObj) jmsOperations.receiveAndConvert();
}
```

### 创建消息驱动的 POJO

**配置消息监听器**

```xml
<!-- 首先把 POJO 注册为 bean -->
<bean id="pojoHandler" class="me.example.PojoHandler"/>

<!-- 把 POJO 声明为消息监听器 -->
<jms:listener-container connection-factory="connectionFactory">
    <jms:listener destination="{destName}" ref="pojoHandler" method="{methodName}"/>
</jms:listener-container>
```

### 使用基于消息的 RPC

**服务端**

```xml
<bean id="myService" class="me.example.MyServiceImpl"/>
<bean id="myServiceExporter" class="org.springframework.jms.remoting.JmsInvokerServiceExporter">
	<property name="service" ref="myService"/>
    <property name="serviceInterface" value="me.example.MyService"/>
</bean>
<jms:listener-container connection-factory="connectionFactory">
	<jms:listener destination="{destName}" ref="myServiceExporter"/>
</jms:listener-container>
```

**客户端**

```xml
<bean id="myService" class="org.springframework.jms.remoting.JmsInvokerProxyFactoryBean">
	<property name="connectionFactory" ref="connectionFactory"/>
    <property name="queueName" value="{destName}"/>
    <property name="serviceInterface" value="me.example.MyService"/>
</bean>
```

## 使用 AMQP 实现消息功能

### 配置 Spring 支持 AMQP 消息

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:rabbit="http://www.springframework.org/schema/rabbit"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://www.springframework.org/schema/beans
		http://www.springframework.org/schema/beans/spring-beans.xsd
		http://www.springframework.org/schema/rabbit
		http://www.springframework.org/schema/rabbit/spring-rabbit-1.0.xsd">
	<!-- 默认 5672 端口，用户名密码均为 guest -->
	<rabbit:connection-factory id="connectionFactory"
		host="{host}" port="{port}" username="{user}" password="{pwd}"/>
</beans>
```

rabbit 命名空间里用来声明队列、exchange 和 binding 的元素：

|元素                             |作用                         |
|:--------------------------------|:----------------------------|
|`<queue>`                        |队列                         |
|`<fanout-exchange>`              |fanout 类型的 exchange       |
|`<header-exchange>`              |header 类型的 exchange       |
|`<topic-exchange>`               |topic 类型的 exchange        |
|`<direct-exchange>`              |direct 类型的 exchange       |
|`<bindings><binding/></bindings>`|exchange 和队列之间的 binding|

以上配置元素要与 `<admin>` 一起使用，它会创建一个 RabbitMQ 管理组件，用来自动创建上述元素声明的对象。

### 使用 RabbitTemplate 发送消息

```xml
<!-- exchange 和 routing-key 可为空 -->
<rabbit:template id="rabbitTemplate"
    connection-factory="connectionFactory"
    exchange="${exchangeName}"
    routing-key="${routingKey}" />
```

```java
public class MyRabbitMQClient {
    @Autowired
    private RabbitTemplate rabbit;

    public void send(MyObj obj) {
        //exchangeName 和 routingKey 缺省为上面配置的值
        rabbit.convertAndSend("{exchangeName}", "{routingKey}", obj);
    }
}
```

### 接受 AMQP 消息

#### 使用 RabbitTemplate 来接受消息

```java
Message msg = rabbit.receive("{queueName}");
//或
MyObj obj = rabbit.receiveAndConvert("{queueName}");
```

其中 {queueName} 也可以在 rabbitTemplate 中注入默认值。

队列中无消息时会立即返回 null，所以要自行管理轮询的频率等细节。

#### 定义消息驱动的 AMQP POJO

```java
@Component
public class MyObjHandler {
    public void handleMyObj(MyObj obj) {
        //TODO
    }
}
```

```xml
<!-- 以下元素都来自 rabbit 命名空间 -->
<listener-container connection-factory="connectionFactory">
    <!-- 多个 queueName 用逗号分隔 -->
    <listener ref="myObjListener" method="handleMyObj" queueNames="${queueName}"/>
    <!-- 也可以用 queue 的 beanId 来引用 -->
    <listener ref="myObjListener" method="handleMyObj" queues="${queueId}"/>
</listener-container>
```

# 第 18 章 使用 WebSocket 和 STOMP 实现消息功能

## 使用 Spring 的低层级 WebSocket API

需要引入 spring-websocket 模块。

需要继承 `WebSocketHandler` 接口，或继承 `AbstractWebSocketHandler` 类。

进行配置，让 Spring 把消息转发给处理类。

```java
@EnableWebSocket
public class WebSocketConfig implements WebSocketConfigurer {
    @Override
    public void registerWebSocketHandlers(WebSocketHandlerRegistry registry) {
        registry.addHandler(exampleHandler(), "/example")
        //.setHandshakeHandler(null); //可以自定义握手处理类
        //.addInterceptors(null);     //可以自定义拦截器
          .setAllowedOrigins("*");    //若不配置来源白名单的话，默认只允许来自当前域的连接
        //.withSockJS();              //用 sockJS 在不支持 websocket 环境下的模拟 websocket
    }

    @Bean
    public ExampleHandler exampleHandler() {
        return new ExampleHandler();
    }
}
```

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:websocket="http://www.springframework.org/schema/websocket"
    xsi:schemaLocation="
        http://www.springframework.org/schema/websocket
        http://www.springframework.org/schema/websocket/spring-websocket.xsd
        http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">

    <websocket:handlers>
        <websocket:mapping handler="macroHandler" path="/marco" />
    </websocket:handlers>

    <bean id="exampleHandler" class="me.example.ExampleHandler"/>

</beans>
```

## 启用 STOMP 消息功能

```java
@Configuration
@EnableWebSocketMessageBroker
public class WebSocketStompConfig implements WebSocketMessageBrokerConfigurer {
  @Override
  public void registerStompEndpoints(StompEndpointRegistry registry) {
    //客户端在发布或订阅消息前要连接到此端点
    registry.addEndpoint("/example").withSockJS();
  }

  @Override
  public void configureMessageBroker(MessageBrokerRegistry registry) {
    //以下前缀会路由到 {@link SimpleBrokerMessageHandler}，这是一个模拟了 STOMP 消息代理的、基于内存的消息代理
    registry.enableSimpleBroker("/queue", "/topic");
    //以下前缀会路由到 {@link @MessageMapping} 注解的方法中
    registry.setApplicationDestinationPrefixes("/app");
    //以下前缀会路由到 STOMP 代理中
    //STOMP 代理中继默认会假设代理监听的是 localhost:61613
    registry.enableStompBrokerRelay("/topic", "/queue");
    //.setRelayHost("");//也可以额外设置一些参数
    //...
  }
}
```

### 处理来自客户端的 STOMP 消息

```java
@Controller
public class ExampleController {
    @MessageMapping("/example")
    public void handleExample(Example example) {
        //...
    }
}
```

Spring 4.0 提供了几个消息转换器

- `ByteArrayMessageConverter`：application/octet-stream <-> byte[]
- `MappingJackson2MessageConverter`：application/json <-> Java 对象
- `StringMessageConverter`：text/plain <-> String

**处理订阅**

```java
@SubscribeMapping({"/subs"})
public Example handleSubscription() {
    return new Example();
}
```

**Javascript 客户端**

```javascript
var url = 'http://host:port/contextpath/endpoint';
var sock = new SockJS(url);
var stomp = Stomp.over(sock);
var payload = JSON.stringify({'message':'Marco!'});
stomp.connect('guest', 'guest', function(frame) {
  stomp.send("/marco", {}, payload);// 第二个参数是头信息的 map
});
```

`@SubscribeMapping` 的主要应用场景是请求-回应模式。这种模式跟 HTTP GET 的请求-响应模式很像，不过是异步的。

### 发送消息到客户端

**在处理消息之后发送消息**

```java
/**
 * 发送一条返回消息到指定目的地，所有订阅该目的地的客户端可以获取这条返回消息
 */
@MessageMapping("/example")
@SendTo("/custom/dest") //不加这个注解的话默认会发到 /topic/example
public Example handleExample(Example incoming) {
    Example outgoing = new Example();
    outgoing.setMessage("blahblah");
    return outgoing;
}

/**
 * 返回消息会直接发给客户端，而不会经过消息代理。
 * 也可以添加 {@code SendTo} 注解，这样就会经过消息代理。
 */
@SubscribeMapping("/subs")
public Example handleSubscription() {
    Example outgoing = new Example();
    outgoing.setMessage("blahblah");
    return outgoing;
}
```

**在应用的任意地方发送消息**

服务端把消息发布到一个主题上

```java
@Service
public class SpittleFeedServiceImpl implements SpittleFeedService {
  // 配置 Spring 开启 STOMP 时就已经在上下文里创建了这个 bean
  @Autowired
  private SimpMessageSendingOperations messaging;

  public void broadcastSpittle(Spittle spittle) {
    messaging.convertAndSend("/topic/spittlefeed", spittle);
  }
}
```

客户端订阅一个 STOMP 主题

```javascript
var sock = new SockJS('spittr');
var stomp = Stomp.over(sock);

stomp.connect('guest', 'guest', function(frame) {
  stomp.subscribe("/topic/spittlefeed", handleSpittle);
});

function handleSpittle(incoming) {
  var spittle = JSON.parse(incoming.body);
  // do something else
}
```

## 为目标用户发送消息

在使用 Spring 和 STOMP 消息功能时，有三种方式利用认证用户：

- `@MessageMapping` 和 `@SubscribeMapping` 标注的方法能使用 `Principal` 来获取认证用户。
- `@MessageMapping`、`@SubscribeMapping` 和 `@MessageException` 方法返回的值能以消息的形式发送给认证用户。
- `SimpMessagingTemplate` 能发送消息给特定用户。

### 在控制器中处理用户的消息

客户端订阅一个用户特定的目的地：

```javascript
stomp.subscribe("/user/queue/notifications", handleNotifications);
```

在内部，以“/user”作为前缀的目的地会通过 `UserDestinationMessageHandler` 进行处理。它会去掉“/user”前缀，并基于用户会话添加一个后缀，例如把“/user/queue/notifications”的订阅路由到“/queue/notifications-user1234abcd”上。

在服务端把返回消息发布到用户特定的目的地：

```java
@MessageMapping("/spittle")
@SendToUser("/queue/notifications") // 会发布到 /queue/notifications-user1234abcd
public Notification handleSpittle(Principal principal, SpittleForm form) {
  Spittle spittle = new Spittle(principal.getName(), form.getText(), new Date());
  spittleRepo.save(spittle);
  return new Notification("Saved spittle");
}
```

如果用户已经认证过的话，会根据 STOMP 帧上的头信息得到 `Principal` 对象。

### 为指定用户发送消息

```java
@Service
public class SpittleFeedServiceImpl implements SpittleFeedService {
  @Autowired
  private SimpMessagingTemplate messaging;
  private Pattern pattern = Pattern.compile("\\@(\\S+)");

  public void broadcastSpittle(Spittle spittle) {
    messaging.convertAndSend("/topic/spittlefeed", spittle);
    // 发送给指定用户
    Matcher matcher = pattern.matcher(spittle.getMessage());
    if (matcher.find()) {
      String username = matcher.group(1);
      // 会发到 /user/tom/queue/notifications 上
      messaging.convertAndSendToUser(username, "/queue/notifications", new Notification("blabla"));
    }
  }
}
```

## 处理异常信息

```java
@MessageExceptionHandler({ExceptionA.class, ExceptionB.class}) // 标注处理异常的方法，可指定若干种异常
@SendToUser("/queue/errors") // 可以将异常信息发送给指定用户
public SpittleException handleException(Throwable throwable) {
  logger.error("blabla");
  return new SpittleException("blabla", throwable);
}
```

# 第 19 章 使用 Spring 发送 Email

# 第 20 章 使用 JMX 管理 Spring Bean

JMX

> Java Management Extensions

使用 JMX 管理应用的核心组件是托管 bean （managed bean，MBean），即暴露特定方法的 JavaBean，这些方法定义了管理接口。

JMX 规范定义了 4 种类型的 MBean：

- 标准 MBean：标准 MBean 的管理接口是通过在固定的接口上执行反射确定的，bean 类会实现这个接口；
- 动态 MBean：动态 MBean 的管理接口是在运行时通过调用 `DynamicMBean` 接口的方法来确定的。因为管理接口不是通过静态接口定义的，因此可以在运行时改变；
- 开放 MBean：开放 MBean 是一种特殊的动态 MBean，其属性和方法只限定于原始类型、原始类型的包装类以及可以分解为原始类型或原始类型包装类的任意类型；
- 模型 MBean：模型 MBean 也是一种特殊的动态 MBean，用于充当管理接口与受管资源的中介。模型 Bean 并不像它们所声明的那样来编写。它们通常通过工厂生成，工厂会使用元信息来组装管理接口。

## 将 Spring bean 导出为 MBean

Spring 的 `MBeanExporter` 可以把 Spring bean 导出为 MBean 服务器内的模型 MBean。MBean 服务器（也称为 MBean 代理）是 MBean 生存的容器。对 MBean 的访问也是通过 MBean 服务器来实现的。

声明一个 `MBeanExporter`

```Java
@Bean
public MBeanExporter mbeanExporter(SpittleController spittleController) {
	MBeanExporter exporter = new MBeanExporter();
	Map<String, Object> beans = new HashMap<>();
	beans.put("spitter:name=SpittleController", spittleController);
	exporter.setBeans(beans);
	return exporter;
}
```

`MBeanExporter` 会假设它正在一个应用服务器中或提供 MBean 服务器的其他上下文中运行。若运行的是独立的应用或运行的容器没有提供 MBean 服务器，就需要在 Spring 上下文中配置一个 MBean 服务器。

在 XML 中使用 `<context:mbean-server>` 元素。或者声明一个 `MBeanServerFactoryBean` bean。可以将该 bean 装配到 `MBeanExporter` 的 `server` 属性中来指定 MBean 暴露到哪个 MBean 服务器中。

将 Spring bean 导出为 JMX MBean 之后，可以使用基于 JMX 的管理工具（如 JConsole 或 VisualVM）查看正在运行的应用，显示 bean 的属性并调用 bean 的方法。

为了对 MBean 的属性和操作获得更细粒度的控制，Spring 提供了几种选择：

- 通过名称来声明需要暴露或忽略的 bean 方法；
- 通过为 bean 增加接口来选择要暴露的方法；
- 通过注解标注 bean 来标识托管的属性和操作。

### 通过名称暴露方法

MBean 信息装配器（MBean info assembler）是限制哪些方法和属性将在 MBean 上暴露的关键。

其中一个 MBean 信息装配器是 `MethodNameBasedMBeanInfoAssembler`。另一个基于方法名称的装配器是 `MethodExclusionMBeanInfoAssembler`。它是前者的反操作，指定了不需要暴露为 MBean 托管操作的方法名称列表。

```java
@Bean
public MethodNameBasedMBeanInfoAssembler assembler1() {
	MethodNameBasedInfoAssembler assembler = new MethodNameBasedMBeanInfoAssembler();
	assembler.setManagedMethods(new String[] {
		"getSpittlesPerPage", "setSpittlesPerPage"
	});
	return assembler;
}

@Bean
public MethodExclusionMBeanInfoAssembler assembler2() {
	MethodExclusionMBeanInfoAssembler assembler = new MethodExclusionMBeanInfoAssembler();
	assembler.setIgnoreMethods(new String[] {
		"spittles"
	});
	return assembler;
}

/**
 * 将装配器装配进 MBeanExporter 中
 */
@Bean
public MBeanExporter mbeanExporter(SpittleController spittleController, MBeanInfoAssembler assembler) {
	MBeanExporter exporter = new MBeanExporter();
	Map<String, Object> beans = new HashMap<String, Object>();
	beans.put("spitter:name=SpittleController", spittleController);
	exporter.setBeans(beans);
	exporter.setAssembler(assembler);
	return exporter;
}
```

### 使用接口定义 MBean 的操作和属性

Spring 的 `InterfaceBasedMBeanInfoAssembler` 是另一种 MBean 信息装配器。通过使用接口选择 bean 的哪些方法需要暴露为 MBean 的托管操作。

假设有如下接口：

```java
public interface SpittleControllerManagedOperations {
	int getSpittlesPerPage();
	void setSpittlesPerPage(int spittlesPerPage);
}
```

只需用如下的 bean 替换掉之前基于方法名的装配器即可：

```java
@Bean
public InterfaceBasedMBeanInfoAssembler assembler() {
	InterfaceBasedMBeanInfoAssembler assembler = 
		new InterfaceBasedMBeanInfoAssembler();
	assembler.setManagedInterfaces(
		new Class<?>[] {SpittleControllerManagedOperations.class}
	);
	return assembler;
}
```

被托管的实现类不是必须实现 `SpittleControllerManagedOperation` 接口的。这个接口只是为了标识导出的内容。不过为了保证 MBean 和实现类之间保持一致，最好还是让实现类实现该接口。

### 使用注解驱动的 MBean

Spring 还提供另一种装配器 `MetadataMBeanInfoAssembler`，它可以使用注解标识哪些 bean 的方法需要暴露为 MBean 的托管操作和属性。

手动装配这种装配器很繁杂，简单的做法是使用 `<context:mbean-exporter>` 元素，用这种装配器代替之前的 `MBeanExporter`。

```xml
<context:mbean-exporter server="mbeanServer" />
```

```java
@Controller
//将 SpitteController 导出为 MBean
//objectName 标识了域（Spitter）和 MBean 的名称
@ManagedResource(objectName="spitter:name=SpittleController")
public class SpittleController {
	...
	//将 spittlesPerPage 暴露为托管属性
	@ManagedAttribute
	public void setSpittlesPerPage(int spittlesPerPage) {
		this.spittlesPerPage = spittlesPerPage;
	}

	@ManagedAttribute
	//不会把属性暴露为 MBean 的托管属性，因为此注解是严格限制方法的。
	//@ManagedOperation
	public int getSpittlePerPage() {
		return spittlePerPage;
	}
}
```

### 处理 MBean 冲突

Spring 提供了 3 种借助 `registrationBehaviorName` 属性来处理 MBean 名字冲突的机制：

- `FAIL_ON_EXISTING`：如果已存在相同名字的 MBean，则失败（默认）
- `IGNORE_EXISTING`：忽略冲突，不注册新的 MBean
- `REPLACING_EXISTING`：用新的覆盖旧的 MBean

```java
@Bean
pubilc MBeanExporter mbeanExporter(
	SpittleController spittleController,
	MBeanInfoAssembler assembler) {
	MBeanExporter exporter = new MBeanExporter();
	Map<String, Object> beans = new HashMap<>();
	beans.put("spitter:name=SpittleController", spittleController);
	exporter.setBeans(beans);
	exporter.setAssembler(assembler);
	exporter.setRegistrationPolicy(RegistrationPolicy.IGNORE_EXISTING);
	return exporter;
}
```

## 远程 MBean

为了满足以标准方式进行远程访问 JMX 的需求，JCP（Java Community Process）制订了 JSR-160:Java 管理扩展远程访问 API 规范（Java Management Extensions Remote API Specification）。该规范定义了 JMX 远程访问的标准，该标准至少需要绑定 RMI 和可选的 JMX 消息协议（JMX Messaging Protocol, JMXMP）。

### 暴露远程 MBean

```java
@Bean
public ConnectorServerFactoryBean connectorServerFactoryBean() {
	ConnectorServerFactoryBean csfb =
		new ConnectorServerFactoryBean();
	//（可选）绑定到一个 RMI 注册表
	csfb.setServiceUrl(
		"service:jmx:rmi://localhost/jndi/rmi://localhost:1099/spitter");
	return csfb;
}
```

`ConnectorServerFactoryBean` 会创建和启动 `JSR-160 JMXConnectorServer`。服务器默认使用 JMXMP 协议并监听 9875 端口，即绑定 `service:jmx:jmxmp://localhost:9875`。

### 访问远程 MBean

//TODO

### 代理 MBean

//TODO

### 处理通知

//TODO

### 监听通知

//TODO

# 第 21 章 借助 Spring Boot 简化 Spring 开发

## Spring Boot 简介

### 添加 Starter 依赖

|Starter                     |所提供的依赖                                                                 |
|:---------------------------|:----------------------------------------------------------------------------|
|spring-boot-starter-actuator|spring-boot-starter、spring-boot-actuator、spring-core                       |
|spring-boot-starter-amqp    |spring-boot-starter、spring-boot-rabbit、spring-core、spring-tx              |
|spring-boot-starter-aop     |spring-boot-starter、spring-aop、AspectJ Runtime、AspectJ Weaver、spring-core|
|spring-boot-starter-batch   |spring-boot-starter、HSQLDB、spring-jdbc、spring-batch-core、spring-core     |

//TODO

## 通过 Acutator 获取了解应用内部状况

```groovy
compile("org.springframework.boot:spring-boot-starter-actuator")
```

```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-actuator</artifactId>
</dependency>
```

- `GET /autoconfig` 描述了 Spring Boot 在使用自动配置的时候所做出的决策；
- `GET /beans` 列出运行应用所配置的 bean；
- `GET /configprops` 列出应用中能够用来配置 bean 的所有属性及其当前的值；
- `GET /dump` 列出应用的线程，包括每个线程的栈跟踪信息；
- `GET /env` 列出应用上下文中所有可用的环境和系统属性变量；
- `GET /env/{name}` 展现某个特定环境变量和属性变量的值；
- `GET /health` 展现当前应用的健康状况；
- `GET /info` 展现应用特定的信息；
- `GET /metrics` 列出应用相关的指标，包括请求特定端点的运行次数；
- `GET /metrics/{name}` 展现应用特定指标项的指标状况；
- `POST /shutdown` 强制关闭应用；
- `GET /trace` 列出应用最近请求相关的元数据，包括请求和相应头。
