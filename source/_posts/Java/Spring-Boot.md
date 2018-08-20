---
title: Spring-Boot
date: 2018-07-10 12:00:00
tags: [Java, Spring]
---

# Spring Boot 预置的依赖

位于 `spring-boot-dependencies-{version}.pom` 文件中。可在用户项目的 pom 中重写特定依赖的版本。

# 属性加载顺序

按以下顺序优先级由高到低。

1. 命令行参数。
2. SPRING_APPLICATION_JSON 中的属性。这是以 JSON 格式配置在系统环境变量中的内容。
3. java:comp/env 中的 JNDI 属性。
4. Java 的系统属性，可以通过 `System.getProperties()` 获得的内容。
5. 操作系统的环境变量。
6. 通过 `random.*` 配置的随机属性。
7. 位于当前应用 jar 包之外，针对不同 `{profile}` 环境的配置文件内容。例如 `application-{profile}.properties` 或是 YAML 定义的配置文件。
8. 位于当前应用 jar 包之内，针对不同 `{profile}` 环境的配置文件内容。例如 `application-{profile}.properties` 或是 YAML 定义的配置文件。
9. 位于当前应用 jar 包之外的 `application.properties` 和 YAML 配置内容。
10. 位于当前应用 jar 包之内的 `application.properties` 和 YAML 配置内容。
11. 在 `@Configuration` 注解修改的类中，通过 `@PropertySource` 注解定义的属性。
12. 应用默认属性，使用 `SpringApplication.setDefaultProperties` 定义的内容。

# 初始化

```java
@SpringBootApplication
public class ReadingListApplication {
  public static void main(String[] args) {
    SpringApplication.run(ReadingListApplication.class, args);
  }
}
```

`@SpringBootApplication`：整合了 `@Configuration`、`@EnableAutoConfiguration`、`@Component`。

- `@EnableAutoConfiguration`：允许 Spring 进行智能配置。Spring 会在用户定义的 bean 注册之后，根据当前环境自动注册一些 bean。通过引入的 `AutoConfigurationImportSelector` 的进行筛选。
- `@Component(excludeFilters = {@Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class), @Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class)})`：启用自动扫描，范围是当前类所在的包。会排除一些组件：
  - 自动扫描用户实现的 `TypeExcludeFilter` 来排除某些类。
  - 排除一些自动配置类，具体在 `META-INF/spring.factories` 中的 `org.springframework.boot.autoconfigure.EnableAutoConfiguration` 中。

# 条件化注解

- `@ConditionalOnBean`、`@ConditionalOnMissingBean`：（没）配置了某个特定的 bean。
- `@ConditionalOnClass`、`@ConditionalOnMissingClass`：Classpath 里（没）有指定的类。
- `@ConditionalOnExpression`：给定的 SpEL 结果为 true。
- `@ConditionalOnJava`：Java 版本匹配某个特定值或范围。
- `@ConditionalOnJndi`：参数中给定的 JNDI 位置必须存在一个，如果没有参数，则要有 JNDI `InitialContext`。
- `@ConditionalOnProperty`：指定的配置属性要有一个明确的值。
- `@ConditionalOnResource`：Classpath 里有指定的资源。
- `@ConditionalOnWebApplication`、`@ConditionalOnNotWebApplication`：这（不）是一个 Web 程序。

# 自定义配置
