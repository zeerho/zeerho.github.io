---
title: Spring 配置文件详解
date: 2017-01-01 09:00:00
tags: [Java]
---

# Spring2.5 配置文件详解
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans  
    xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
    xmlns:context="http://www.springframework.org/schema/context"
    xmlns:aop="http://www.springframework.org/schema/aop"
    xmlns:tx="http://www.springframework.org/schema/tx"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
                        http://www.springframework.org/schema/beans/spring-beans-2.5.xsd
                        http://www.springframework.org/schema/context
                        http://www.springframework.org/schema/context/spring-context-2.5.xsd
                        http://www.springframework.org/schema/aop
                        http://www.springframework.org/schema/aop/spring-aop-2.5.xsd
                        http://www.springframework.org/schema/tx
                        http://www.springframework.org/schema/tx/spring-tx-2.5.xsd"
>
	<context:annotation-config/>
	<context:component-scan base-package="com.demo"/>
	<aop:aspectj-autoproxy/>
	
	<bean id="Bean实例名称" class="全限定类名"/>
	<bean id="Bean实例名称" class="全限定类名" scope="prototype"/>
	<bean id="Bean实例名称" class="全限定类名"
		init-method="初始化时调用的方法名"
		destroy-method="对象销毁时调用的方法名"/>
	<bean id="Bean实例名称" class="全限定类名">
		<property name="属性名" ref="要引用的Bean的id"/>
		<property name="属性名" value="直接指定属性值"/>
		<property name="属性名">
			<bean class="全限定类名"/>
		</property>
		<property name="Bean中Set类型属性名称">
			<set>
				<value>set中的元素</value>
				<ref bean="要引用的Bean的id"/>
			</set>
		</property>
		<property name="Bean中List类型属性名称">
			<list>
				<value>list中的元素</value>
				<ref bean="要引用的Bean的id"/>
			</list>
		</property>
		<property name="Bean中Map类型属性名称">
			<map>
				<entry key="Map元素的key">
					<value>Map元素的value</value>
				</entry>
				<entry key="Map元素的key">
					<ref bean="要引用的Bean的id"/>
				</entry>
			</map>
		</property>
		<property name="Bean中Properties类型属性名称">
			<props>
				<prop key="properties元素的key">properties元素的value</prop>
			</props>
		</property>
		<property name="Bean中要初始化为null的属性名称">
			<null/>
		</property>
	</bean>
	<bean id="Bean实例名称" class="全限定类名">
		<constructor-arg index="从0开始的序号" type="构造参数的类型" value="构造参数的值"/>
		<constructor-arg index="从0开始的序号" type="构造参数的类型" ref="要引用的Bean的id"/>
		
		
	</bean>	
	<bean id="目标对象名称" class="目标对象全限定类名"/>
	<bean id="切面实例名称" class="切面类全限定类名"/>
	
	<aop:config>
		<aop:aspect id="切面id" ref="要引用的切面实例id">
			<aop:pointcut id="切点名称" expression="切点正则表达式"/>
			<aop:before pointcut-ref="切点名称" method="切面类中用作前置通知的方法名"/>
			<aop:after-returning pointcut-ref="切点名称" method="切面类中用作后置通知的方法名"/>
			<aop:after-throwing pointcut-ref="切点名称" method="切面类中用作异常通知的方法名"/>
			<aop:after pointcut-ref="切点名称" method="切面类中用作最终通知的方法名"/>
			<aop:around pointcut-ref="切点名称" method="切面类中用作环绕通知的方法名"/>
		</aop:aspect>
	</aop:config>
	
	<!--配置事务管理器-->
	<bean id="事务管理器实例名称" class="事务管理器全限定类名">
		<property name="数据源属性名称" ref="要引用的数据源实例名称"/>
	</bean>
	<!--配置一个事务通知-->
	<tx:advice id="事务通知名称" transaction-manager="事务管理器实例名称">
		<tx:attributes>
			<!--方法以get开头的，不用事务-->
			<tx:method name="get*" read-only="true" propagation="NOT_SUPPORTED"/>
			<!--其他方法以默认事务进行-->
			<tx:method name="*"/>
		</tx:attributes>
	</tx:advice>
	<!--使用AOP技术实现事务管理-->
	<aop:config>
		<aop:pointcut id="事务切点名称" expression="事务切点正则表达式"/>
		<aop:advisor advice-ref="事务通知名称" pointcut-ref="事务切点名称"/>
	</aop:config>
</beans>
```
![Spring配置详解](http://7xvqyt.com1.z0.glb.clouddn.com/Spring%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6%E8%AF%A6%E8%A7%A3.bmp)
