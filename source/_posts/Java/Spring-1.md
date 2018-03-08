---
title: Spring-1
date: 2017-01-01 09:00:01
tags: [Java, Spring]
---

整理自《Spring实战（第4版）》

**第 1 部分 Spring 的核心**

<!-- more -->

# 第 1 章 Spring 之旅

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

```java
@Bean
public static PropertySourcesPlaceholderConfigurer placeHolderConfigurer() {
	return new PropertySourcesPlaceholderConfigurer();
}
```

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
	@Value("${disc.artist") String artist) {
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
`#{sgtPeppers}` bean 的属性
`#{artistSelector.selectArtist()}` 调用 bean 的方法
`#{artistSelector.selectArtist()?.toUpperCase()}` 调用返回值的方法。`?` 会进行判空，为空则直接返回 null，因而不会空指针异常

**在表达式中使用类型**

`#{T(java.lang.Math).PI}` 访问类的静态成员
`#{T(java.lang.Math).random()}` 访问类的静态方法

**SpEL 运算符**

|运算符类型|运算符                              |
|:---------|:-----------------------------------|
|算术运算  |+、-、*、/、%、^                    |
|比较运算  |<、>、==、<=、>=、lt、gt、eq、le、ge|
|逻辑运算  |and、or、not、\|                    |
|条件运算  |?: (ternary)、?: (Elvis)            |
|正则表达式|matches                             |

`#{scoreboard.score > 1000 ? "Winner" : "Loser"}` 三元运算
`#{disc.title ?: 'Default Title'}` 为 null 的话取默认值

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
