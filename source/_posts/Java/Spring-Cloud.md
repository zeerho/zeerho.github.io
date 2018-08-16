---
title: Spring-Cloud
date: 2018-08-10 12:00:00
tags: [Java, Spring]
---

# Spring Cloud 和 Spring Boot 的兼容关系

|Spring Cloud   |Spring Boot   |
|:--------------|:-------------|
|Finchley       |(1.5.x, 2.0.x]|
|Dalston,Edgware|[1.5.x, 2.0.x)|
|Camden         |[1.4.x, 1.5.x]|
|Brixton        |[1.3.x, 1.4.x]|
|Angel          |1.2.x         |

# actuator

## 原生端点

应用配置类

- `/autoconfig`：包括所有自动化配置的候选项。同时还有每个候选项是否满足自动化配置的各个先决条件。
  - `positiveMatches` 中返回的是条件匹配成功的自动化配置。
  - `negativeMatches` 中返回的是条件匹配不成功的自动化配置。
- `/beans`：用来获取应用上下文中创建的所有 bean。
- `/configprops`：用来获取应用中配置的属性信息报告。
- `/env`：用来获取应用所有可用的环境属性报告。包括环境变量、JVM 属性、应用的配置属性、命令行中的参数。
- `/mappings`：用来返回所有 Spring MVC 的控制器映射关系报告。
- `/info`：用来返回一些应用自定义的信息。这些信息要在 `application.properties` 配置文件中通过 `info` 前缀来设置。

度量指标类

- `/metrics`：用来返回当前应用的各类重要度量指标，比如内存信息、线程信息、垃圾回收信息等。还可以通过 `/metrics/{name}` 接口来获取细粒度的信息。
- `/health`：spring-boot-starter-actuator 模块中自带了一些健康指标检测器，这些检测器都是 `HealthIndicator` 接口的实现。
  - `DiskSpaceHealthIndicator`：低磁盘空间检测。
  - `DataSourceHealthIndicator`：检测 DataSource 的连接是否可用。
  - `MongoHealthIndicator`：检测 Mongo 服务器是否可用。
  - `RabbitHealthIndicator`：检测 Rabbit 服务器是否可用。
  - `RedisHealthIndicator`：检测 Redis 服务器是否可用。
  - `SolrHealthIndicator`：检测 Solr 服务器是否可用。
- `/dump`：用来暴露程序运行中的线程信息。它使用 `java.lang.management.ThreadMXBean` 的 `dumpAllThreads` 方法来返回所有含有同步信息的活动线程详情。
- `/trace`：用来返回基本的 HTTP 跟踪信息。

操作控制类

- `/shutdown`：原生端点只有这个，并且需要配置开启：`endpoints.shutdown.enabled=true`。

# 服务治理：Spring Cloud Eureka

```xml
<dependencies>
  <dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
  <dependency>
<dependencies>

<dependencyManagement>
	<dependencies>
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-dependencies</artifactId>
			<version>Finchley.SR1</version>
			<type>pom</type>
			<scope>import</scope>
		</dependency>
	</dependencies>
</dependencyManagement>
```

```java
@EnableEurekaServer // 启动服务注册中心
@SpringBootApplication
public class Application {
  public static void main(String[] args) {
    new SpringApplicationBuilder(Application.class).web(true).run(args);
  }
}
```

默认设置下，服务注册中心也会把自己作为客户端来注册自己。可以在 `application.properties` 中禁用：

```properties
 # 不向注册中心注册自己
eureka.client.register-with-eureka=false
 # 不检索服务
eureka.client.fetch-registry=false
```
