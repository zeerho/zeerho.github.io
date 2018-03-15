---
title: Effective Java 笔记
date: 2017-01-01 09:00:00
tags: [Java]
---

# 创建和销毁对象

## 1 考虑用静态工厂方法代替构造器

1. 静态工厂方法可以自定义名称，这样可以根据方法名来区分不同的对象实例化入口。而构造器只能通过参数来区分，容易弄错。
2. 静态工厂方法可以灵活地决定返回新对象还是单例。
3. 静态工厂方法可以返回原返回类型的任何子类型。
4. 静态工厂方法在创建参数化类型实例时可以利用编译器的类型推导特性来使代码更简洁（Java 8 中构造器也实现了这个特性）。

静态工厂方法返回的对象所属的类，在编写该静态工厂方法时可以不必存在。这种灵活的静态工厂方法构成了**服务提供者框架（Service Provider Framework）**。

**服务提供者框架**
> 多个服务提供者实现一个服务，系统为服务提供者的客户端提供多个实现，并把它们从多个实现中解耦出来。

服务提供者框架中的三个重要组件和一个可选组件：

- 服务接口（Service Interface）：由提供者实现；
- 提供者注册 API（Provider Registration API）：系统用来注册实现或提供者，让客户端访问它们；
- 服务访问 API（Service Access API）：客户端用来获取服务的实例；
- 服务提供者接口（Service Provider Interface）（可选）：这些提供者负责创建其服务实现的实例。如果没有服务提供者接口，实现就按照类名称注册，并通过反射方式进行实例化。

服务访问 API 是“灵活的静态工厂”，它是服务提供者框架的基础。

```java
//服务接口
public interface Service {
    ...
}

//服务提供者接口
public interface Provider {
    Service newService();
}

//提供服务注册和访问的类，本身不能被实例化
public class Services {
    private Services() {}
    
    private static final Map<String, Provider> providers = new ConcurrentHashMap<String, Provider>();
    private static final String DEFAULT_NAME = "default";
    
    //提供者注册接口
    public static void registerDefaultProvider(Provider p) {
        registerProvider(DEFAULT_NAME, P);
    }
    public static void registerProvider(String name, Provider p) {
    providers.put(name, p);
    }
    
    //服务访问 API
    public static Service newInstance() {
        return newInstance(DEFAULT_NAME);
    }
    public static Service newInstance(String name) {
        Provider p = providers.get(name);
        if (p == null) {
            throw new IllegalArgumentException("blabla");
        }
        return p.newService();
    }
```

## 3 用私有构造器或者枚举类型强化 Singleton 属性

私有构造器只阻断了 new 的构造方式，除此之外还有两种对象实例化方式：

- 反射
- 反序列化

解决方法依次是：

- 在私有构造器中抛出异常。
- 所有的实例域都加上`transient` 修饰；实现 `readResolve()` 方法，方法中直接返回已存在的单例。

Java 5 开始的 enum 可以阻断上述三种实例化方式，这样构造单例就很简单。

## 4 消除过期的对象引用

