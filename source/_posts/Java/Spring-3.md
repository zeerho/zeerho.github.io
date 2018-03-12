---
title: Spring-3
date: 2017-01-01 09:00:03
tags: [Java, Spring]
---

整理自《Spring实战（第4版）》

**第 3 部分 后端中的 Spring**

<!-- more -->

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
