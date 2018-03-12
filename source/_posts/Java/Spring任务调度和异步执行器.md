---
title: Spring任务调度和异步执行器
date: 2018-03-01 09:00:00
tags: [Spring]
---

# Java 5.0 的 Executor

`java.util.concurrent.Executor` 接口将“任务提交”和“任务执行”分离解耦。

它有两个子接口：

- `ExecutorService`：添加了结束任务的管理方法，在提交任务时还可以获取一个 `Future` 实例，通过该实例跟踪异步任务的情况。
    - `TreadPoolExecutor` 实现了 `Executor` 和 `ExecutorService` 两个接口，它使用一个线程池对任务进行调度。
- `ScheduledExecutorService`：可以进行任务调度，比如指定延迟时间和执行周期。
    - `ScheduledThreadPoolExecutor` 是 `ThreadPoolExecutor` 的子类，实现了 `ScheduledExecutorService` 接口，对任务添加了调度功能。

`java.util.concurrent` 中为创建上述接口实例提供了一个工厂类 `Executors`：

- `ExecutorService newFixedThreadPool(in nThreads)`：创建一个线程池；
- `ExecutorService newCachedThreadPool()`：动态线程池，不够用时创建新线程，长时间闲置的线程被回收；
- `ScheduledExecutorService newScheduledThreadPool(int corePoolSize, ThreadFactory threadFactory)`：创建一个线程池，可指定延迟和周期。

Spring 的 `TaskExecutor` 接口等同于 `java.util.concurrent.Executor` 接口。Spring 提供了一些实现类：

- `SyncTaskExecutor`：直接在主线程中执行。
- `SchedulingTaskExecutor`：子接口，提供了调度功能。
    - `SimpleAsyncTaskExecutor`：每次执行创建新线程，但支持对并发总数的限制，超出部分会被阻塞。
    - `ConcurrentTaskExecutor`：是 Java 5.0 的 `Executor` 的适配器。
    - `SimpleThreadPoolTaskExecutor`：是 Quartz 的 `SimpleThreadPool` 的子类，负责监听 Spring 的生命周期回调。当需要在 Quartz 和非 Quartz 组件中共享线程池时使用。
    - `ThreadPoolTaskExecutor`：只能在 Java 5.0 中使用。它暴露了一些属性，方便在 Spring 中配置一个 `java.util.concurrent.ThreadPoolExecutor`，并包装成 `TaskExecutor`。
    - `TimerTaskExecutor`：使用一个 Timer 作为后台实现。


