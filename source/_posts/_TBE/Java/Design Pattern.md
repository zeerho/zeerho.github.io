---
title: Design Pattern
date: 2017-01-01 09:00:00
tags: [Java]
---

# 设计原则

- 封装变化
- 多用组合，少用继承
- 针对接口编程，而不是实现
- 为交互对象之间的松耦合设计而努力
- 类应该对扩展开放，对修改关闭
- 依赖抽象，不要依赖具体类
- 只和朋友交谈
- 别找我，我会找你
- 一个类应该只有一个引起变化的原因


# 最小知识原则

对于任何一个对象，应设计成只跟**尽量少**的其他对象进行交互。

在一个对象的方法内，只应该调用属于以下范围的方法：

- 该对象本身
- 被当作方法的参数而传递进来的对象
- 此方法创建的对象
- 该对象持有的组件

不推荐的方式
```
public float getPrice() {
    Car car = shop.getCar("BMW");
    return car.getPrice();
}
```

推荐的方式
```
public float getPrice() {
    return shop.getPrice("BMW");
}
```

优点：减少对象之间的耦合，减少维护成本。
缺点：产生额外的包装类，增加开发时间，降低运行时性能。

# 好莱坞原则

> 别调用我们，我们会调用你。

系统组件之间应形成高低关系，其中高层组件可以决定是否调用低层组件，而低层组件不能调用高层组件。
这样使得依赖关系形成单调的趋势，而不是循环依赖。

# 策略模式 Strategy

> 策略模式定义了算法族，分别封装起来，让他们之间可以相互替换，此模式让算法的变化独立于使用算法的客户。

# 观察者模式 Subject & Observer

> 定义了对象之间的一对多依赖，这样一来，当一个对象改变状态时，它的所有依赖者都会收到通知并自动更新。

# 装饰者模式 Decorator

> 动态地将责任附加到对象上。若要扩展功能，装饰者提供了比继承更有弹性的替代方案。

# 工厂方法模式 Factory

**工厂方法**
> 使用继承，把对象的创建委托给子类，父类提供抽象的创建方法。

**抽象工厂方法**
> 使用组合，向消费者提供一个工厂接口类型，该接口中含有抽象的创建方法。通过向消费者传入具体的工厂接口实现来得到不同的创建方法。这个工厂接口本身通常就使用了工厂方法的设计模式。

# 单例模式 Singleton

> 确保一个类只有一个实例，并提供一个全局访问点。

单线程中的简单做法，在多线程中会有并发问题

```
public class MyClass {
    private static MyClass myClassSingleton;
    private MyClass() {}
    public static MyClass getInstance() {
        if (myClassSingleton == null) {
            myClassSingleton = new MyClass();
        }
        return myClassSingleton;
    }
}
```

解决并发问题的简单方法，但在频繁调用的情况下会严重影响性能

```
public class MyClass {
    private static MyClass myClassSingleton;
    private MyClass() {}
    public static synchronized MyClass getInstance() {
        if (myClassSingleton == null) {
            myClassSingleton = new MyClass();
        }
        return myClassSingleton;
    }
}
```

“饿汉”模式，在静态初始化器中创建单件，利用 JVM 自身的特性保证了线程安全，在 JVM 加载这个类时就创建了单件实例，从而保证在任何线程访问该实例前就已经创建完毕。
若有大量使用概率很低的单例类使用“饿汉”模式，会造成明显的内存浪费。

```
public class MyClass {
    private static MyClass myClassSingleton = new MyClass();//加载类时创建实例
    private MyClass() {}
    public static MyClass getInstance() {
        return myClassSingleton;
    }
}
```

“饱汉”模式，只在第一次访问单例时创建单例对象。

```
public class MyClass {
    //volatile 关键词确保 myClassSingleton 赋值时能立即对所有线程可见。在 Java 1.4 及之前的版本中 volatile 实现有问题。
    private volatile static MyClass myClassSingleton；
    private MyClass() {}
    public static MyClass getInstance() {
        if ( myClassSingleton == null) { //double check 解决多线程并发问题
            synchronized (MyClass.class) {
                if (myClassSingleton == null) {
                    myClassSingleton = new MyClass();
                }
            }
        }
        return myClassSingleton;
    }
}
```

**若程序中使用了多个类加载器（Class Loader），会产生多个单例**

**在 Java 1.2 之前，GC 有个 bug，当单例对象只被自身的类所引用时，会被 GC 清理**

# 命令模式 Command

> 将“请求”封装成对象，以便使用不同的请求、队列或者日志来参数化其他对象。命令模式也支持可撤销的操作。
本质上是在调用者和接收者之间加了一层用于命令的整合、转换、映射，从而实现调用者和接收者之间的解耦。

# 适配器模式 Adapter

> 将一个类的接口，转换成客户期望的另一个接口，从而让原本不兼容的类之间可以兼容。

# 外观模式 Facade

> 外观模式提供了一个统一的接口，用来方法子系统中的一群接口。外观定义了一个高层接口，让子系统更容易使用

外观模式和适配器模式类似，都是在不同接口方法之间建立映射关系。
其中适配器模式着重**转换**，而外观模式着重于**简化**。

# 模板方法模式 Template

> 模板方法模式在一个方法中定义一个算法的骨架，而将一些步骤延迟到子类中。模板方法使得子类可以在不改变算法结构的情况下，重新定义算法中的某些步骤。

# 迭代器模式 Iterator

> 迭代器模式提供一种方法顺序访问一个聚合对象中的各个元素，而又不暴露其内部的表示。

外部迭代器：客户自己通过调用 next 方法进行游走，并在游走的过程中进行处理。
内部迭代器：游走过程由迭代器自行完成，客户无法干预，且必须将游走过程中需要处理的逻辑传给迭代器。

# 组合模式 Composite

> 组合模式允许你将对象组合成树形结构来表现“整体/部分”层次结构。组合能让客户以一致的方式处理个别对象以及组合对象。

# 状态模式 State

> 状态模式允许对象在内部状态改变时改变它的行为，对象看起来好像修改了它的类。

和策略模式一样，都是在一个接口下用不同的类封装不同的逻辑。区别在于策略模式一般由调用者直接或间接指定某个逻辑；而状态模式在内部进行策略转换。