---
title: Spring-2
date: 2017-01-01 09:00:02
tags: [Java, Spring]
---

整理自《Spring实战（第4版）》

**第 2 部分 Web 中的 Spring**

<!-- more -->

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
