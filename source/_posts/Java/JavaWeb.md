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
