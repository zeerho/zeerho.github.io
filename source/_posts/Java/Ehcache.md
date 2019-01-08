---
title: Ehcache
date: 2019-01-01 20:00:00
tags: [Java]
---

# 配置

**ehcache-setting.xml**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<ehcache>
  <!-- 当 Ehcache 持久化数据时就把数据写到这个目录下 -->
  <diskStore path="java.io.tmpdir"/>

  <!-- 设定缓存的默认数据过期策略 -->
  <defaultCache
    maxElementsInMemory="10000" 
    eternal="false" 
    overflowToDisk="true"
    timeToIdleSeconds="10"
    timeToLiveSeconds="20"
    diskPersistent="false"
    diskExpiryThreadIntervalSeconds="120"/>

  <cache name="cacheTest"
    maxElementsInMemory="1000"
    eternal="false"
    overflowToDisk="true"
    timeToIdleSeconds="10"
    timeToLiveSeconds="20"/>

</ehcache>
```

- name：缓存名称。
- maxElementsInMemory：内存中最大缓存对象数。
- maxElementsOnDisk：硬盘中最大缓存对象数，0 表示无穷大。
- eternal：true 表示对象永不过期，此时会忽略 timeToIdleSeconds 和 timeToLiveSeconds 属性 ，默认为 false。
- overflowToDisk：true 表示当内存缓存的对象数目达到了 maxElementsInMemory 界限后，会把溢出的对象写到硬盘缓存中。注意：如果缓存的对象要写入到硬盘中的话，则该对象必须实现了 `Serializable` 接口才行。
- diskSpoolBufferSizeMB：磁盘缓存区大小，默认为30MB。每个Cache都应该有自己的一个缓存区。
- diskPersistent：是否缓存虚拟机重启期数据，是否持久化磁盘缓存，当这个属性的值为 true 时，系统在初始化时会在磁盘中查找文件名 为 cache 名称，后缀名为 index 的文件，这个文件中存放了已经持久化在磁盘中的cache的index,找到后会把cache加载到内存，要想把 cache真正持久化到磁盘,写程序时注意执行net.sf.ehcache.Cache.put(Element element)后要调用flush()方法。
- diskExpiryThreadIntervalSeconds：磁盘失效线程运行时间间隔，默认为120秒。
- timeToIdleSeconds： 设定允许对象处于空闲状态的最长时间，以秒为单位。当对象自从最近一次被访问后，如果处于空闲状态的时间超过了timeToIdleSeconds属性 值，这个对象就会过期，EHCache将把它从缓存中清空。只有当eternal属性为false，该属性才有效。如果该属性值为0，则表示对象可以无限 期地处于空闲状态。
- timeToLiveSeconds：设定对象允许存在于缓存中的最长时间，以秒为单位。当对象自从被存放到缓存中后，如果处于缓存中的时间超过了 timeToLiveSeconds属性值，这个对象就会过期，EHCache将把它从缓存中清除。只有当eternal属性为false，该属性才有 效。如果该属性值为0，则表示对象可以无限期地存在于缓存中。timeToLiveSeconds必须大于timeToIdleSeconds属性，才有意义。
- memoryStoreEvictionPolicy：当达到maxElementsInMemory限制时，Ehcache将会根据指定的策略去清理内存。可选策略有：LRU（最近最少使用，默认策略）、FIFO（先进先出）、LFU（最少访问次数）。


**Spring 的配置**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
  xmlns:cache="http://www.springframework.org/schema/cache"
  xmlns:context="http://www.springframework.org/schema/context"
  xmlns:aop="http://www.springframework.org/schema/aop"
  xsi:schemaLocation="  
      http://www.springframework.org/schema/beans  
      http://www.springframework.org/schema/beans/spring-beans-3.0.xsd  
      http://www.springframework.org/schema/aop  
      http://www.springframework.org/schema/aop/spring-aop-3.0.xsd
      http://www.springframework.org/schema/context  
      http://www.springframework.org/schema/context/spring-context-3.0.xsd
      http://www.springframework.org/schema/cache 
      http://www.springframework.org/schema/cache/spring-cache-3.1.xsd">

  <!-- 自动扫描注解的bean -->
  <context:component-scan base-package="com.example.service" />

  <cache:annotation-driven cache-manager="cacheManager" />  

  <bean id="cacheManager" class="org.springframework.cache.ehcache.EhCacheCacheManager">  
    <property name="cacheManager" ref="ehcache"></property>  
  </bean>  

  <bean id="ehcache" class="org.springframework.cache.ehcache.EhCacheManagerFactoryBean">  
    <property name="configLocation" value="classpath:ehcache-setting.xml"/>
  </bean>  

</beans>
```

