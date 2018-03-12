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
- `Scheduled


