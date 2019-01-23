---
title: JavaWeb
date: 2018-05-14 16:43:00
tags: [Java]
---

# url-pattern

`<url-pattern>/*</url-pattern>`

匹配所有路径，会覆盖所有其他 servlet 的路径映射，包括 servlet 容器所有内置缺省的 servlet（这些 servlet 一般用来处理静态资源、HTTP 缓存请求、媒体数据流和文件下载）。这种匹配方式一般用在过滤器上。

`<url-pattern>/</url-pattern>`

只覆盖 servlet 容器内置缺省的 servlet，不覆盖其他的任何 servlet。也就是说，要自己实现对静态资源等请求的映射，但这里所说的静态资源不含 jsp 文件，因为 jsp 文件由另一个内置的 servlet 来负责处理。可在主配置中做如下配置：

```xml
<mvc:resources location="somewhere/js/" mapping="/js/**" />
<mvc:resources location="somewhere/css/" mapping="/css/**" />
<mvc:resources location="somewhere/images/" mapping="/images/**" />
<!-- 或者 -->
<mvc:default-servlet-handler />
```

`<url-pattern></url-pattern>`

当上下文的根被请求的时候，它将被调用。这跟 `<welcome-file>` 的方式是不同的，因为这种形式在当任何子目录被请求的时候不会被调用。一般用来处理对于首页的请求。

# load-on-startup

1. `load-on-startup` 元素标记容器是否应该在 web 应用程序启动的时候就加载这个 servlet，(实例化并调用其 `init()` 方法)。
2. 它的值必须是一个整数，表示 servlet 被加载的先后顺序。
3. 如果该元素的值为负数或者没有设置，则容器会当 Servlet 被请求时再加载。
4. 如果值为正整数或者 0 时，表示容器在应用启动时就加载并初始化这个 servlet，值越小，优先级越高，就越先被加载。值相同时，容器就会自己选择顺序来加载。

# HttpServletRequest#getSession()

- `getSession(true)`: 若没有会话则创建。
- `getSession()`: 同上。
- `getSession(false)`: 若没有会话则返回 null。

# HttpSession#getId() 和 HttpServletRequest#getRequestedSessionId()

- `HttpSession#getId()`: web 容器创建会话时创建的 id。
- `HttpServletRequest#getRequestedSessionId()`: 浏览器请求的 id，对应 cookie 中的 JSESSIONID 字段（可以在容器中自定义名称）。可能与内部的 id 不同，比如服务端会话过期后新建了而浏览器的 cookie 还没更新，或浏览器的 cookie 被清理了而服务端还没过期（后者情况会认证失败）。

