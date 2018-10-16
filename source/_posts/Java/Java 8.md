---
title: Java 8
date: 2017-01-01 09:00:00
tags: [Java]
---

# Lamda

<!-- more -->

## 形式

`(param) -> expression` 或 `(param) -> { statements; }`

其中

- 参数的类型可以省略（必须同时（不）省略），前提是可以根据上下文推断出类型。
- 参数名称不能跟当前作用域内的变量冲突。

## 将 lamda 表达式替换为方法引用

*参考并摘自 [developerWorks-传递表达式（pass-through lambdas）的替代方案](https://www.ibm.com/developerworks/cn/java/j-java8idioms5/index.html)。*

若 lamda 中没有做实际的操作，而仅仅是将参数传递给了另一个方法，则最好将 lamda 替换为方法引用。

```
e -> System.out.println(e)
替换为
System.out::println
```

## lamda 表达式中的形参有几种不同的用途：

- 形参作为另一个方法的实参
- 形参作为类/对象，并调用其中的方法
- 传递一个构造函数
- 传递多个实参

### 传递形参作为实参

```
实例方法的实参
variable -> reference.instanceMethod(variable)
reference::instanceMethod

静态方法的实参
variable -> Clazz.staticMethod(variable)
Clazz::staticMethod
```

其中，`this` 也可作为实例
```
variable -> this.instanceMethod(variable)
this::instanceMethod
```

### 传递形参作为目标

```
variable -> variable.instanceMethod()
Clazz::instanceMethod
```

**模糊性与方法引用**
若一个类的一个静态方法和一个兼容的实例方法同名，那么使用方法引用会引起歧义，使得编译失败。
比如将 `(Integer e) -> e.toString` 写为 `Integer::toString`，由于 Integer 类同时包含静态方法 `public static String toString(int i)` 和实例方法 `public String toString()`，所以编译器无法确定该方法引用对应哪个方法。这种情况下只能使用 lamda 表达式来确定唯一的一个方法。

### 传递构造函数调用

```
variable -> new Clazz(variable)
Clazz::new
```

### 传递多个形参

传递多个形参作为实参
```
var1, var2 -> Clazz.methodName(var1, var2)
//前提是形参和实参的顺序相同
Clazz::methodName
```

分别作为目标和实参
```
var1, var2 -> var1.methodName(var2);
//前提是第一个形参作为目标
Clazz:methodName
```

**两种多形参的情况依然需要先保证无歧义的前提条件**

# Java 预设的函数式接口

大致分为 4 类：

- Consumer：有入参，无出参；
- Function：有入参，有出参；
    - Predicate：出参固定为 boolean 的 Function，也可以单独算一类；
- Supplier：无入参，有出参。

| 接口                                        | 主要方法                                        |
| :------------------------------------------ | :---------------------------------------------- |
| Consumer<T>                                 | void accept(T t)                                |
| BiConsumer<T, U>                            | void accept(T t, U u)                           |
| DoubleConsumer                              | void accept(double value)                       |
| IntConsumer                                 | void accept(int value)                          |
| LongConsumer                                | void accept(long value)                         |
| ObjDoubleConsumer<T>                        | void accept(T t, double value)                  |
| ObjIntConsumer<T>                           | void accept(T t, int value)                     |
| ObjLongConsumer<T>                          | void accept(T t, long value)                    |
|                                             |                                                 |
| Function<T, R>                              | R apply(T t)                                    |
| BiFunction<T, U, R>                         | R apply(T t, U u)                               |
| BinaryOperator<T> extends BiFunction<T,T,T> |                                                 |
| DoubleFunction<R>                           | R apply(double value)                           |
| DoubleToIntFunction                         | int applyAsInt(double value)                    |
| DoubleToLongFunction                        | long applyAsLong(double value)                  |
| IntFunction<R>                              | R apply(int value)                              |
| IntToDoubleFunction                         | double applyAsDouble(int value)                 |
| IntToLongFunction                           | long applyAsLong(int value)                     |
| LongFunction<R>                             | R apply(long value)                             |
| LongToDoubleFunction                        | double applyAsDouble(long value)                |
| LongToIntFunction                           | int applyAsInt(long value)                      |
| ToDoubleBiFunction<T, U>                    | double applyAsDouble(T t, U u)                  |
| ToDoubleFunction<T>                         | double applyAsDouble(T value)                   |
| ToIntBiFunction<T, U>                       | int applyAsInt(T t, U u)                        |
| ToIntFunction<T>                            | int applyAsInt(T value)                         |
| ToLongBiFunction<T, U>                      | long applyAsLong(T t, U u)                      |
| ToLongFunction<T>                           | long applyAsLong(T value)                       |
| UnaryOperator<T> extends Function<T, T>     |                                                 |
| DoubleBinaryOperator                        | double applyAsDouble(double left, double right) |
| DoubleUnaryOperator                         | double applyAsDouble(double operand)            |
| IntBinaryOperator                           | int applyAsInt(int left, int right)             |
| IntUnaryOperator                            | int applyAsInt(int operand)                     |
| LongBinaryOperator                          | long applyAsLong(long left, long right)         |
| LongUnaryOperator                           | long applyAsLong(long operand)                  |
|                                             |                                                 |
| Predicate<T>                                | boolean test(T t)                               |
| BiPredicate<T, U>                           | boolean test(T t, U u)                          |
| DoublePredicate                             | boolean test(double value)                      |
| IntPredicate                                | boolean test(int value)                         |
| LongPredicate                               | boolean test(long value)                        |
|                                             |                                                 |
| Supplier<T>                                 | T get()                                         |
| BooleanSupplier                             | boolean getAsBoolean()                          |
| DoubleSupplier                              | double getAsDouble()                            |
| IntSupplier                                 | int getAsInt()                                  |
| LongSupplier                                | long getAsLong()                                |

# 中间操作和终端操作

|操作     |类型               |返回类型   |参数类型              |函数描述符    |
|:--------|:------------------|:----------|:---------------------|:-------------|
|filter   |中间               |Stream<T>  |Predicate<T>          |T -> boolean  |
|distinct |中间（有状态-无界）|Stream<T>  |                      |              |
|skip     |中间（有状态-有界）|Stream<T>  |long                  |              |
|limit    |中间（有状态-有界）|Stream<T>  |long                  |              |
|map      |中间               |Stream<R>  |Function<T, R>        |T -> R        |
|flatMap  |中间               |Stream<R>  |Function<T, Stream<R>>|T -> Stream<R>|
|sorted   |中间（有状态-无界）|Stream<T>  |Comparator<T>         |(T, T) -> int |
|anyMatch |终端               |boolean    |Predicate<T>          |T -> boolean  |
|noneMatch|终端               |boolean    |Predicate<T>          |T -> boolean  |
|allMatch |终端               |boolean    |Predicate<T>          |T -> boolean  |
|findAny  |终端               |Optional<T>|                      |              |
|findFirst|终端               |Optional<T>|                      |              |
|forEach  |终端               |void       |Consumer<T>           |T -> void     |
|collect  |终端               |R          |Collector<T, A, R>    |              |
|reduce   |终端（有状态-有界）|Optional<T>|BinaryOperator<T>     |(T, T) -> T   |
|count    |终端               |long       |                      |              |

# 数值流

## 原始类型流特化

`IntStream Stream#mapToInt(ToIntFunction<T>)`
`LongStream Stream#mapToLong(ToLongFunction<T>)`
`DoubleStream Stream#mapToDouble(ToDoubleFunction<T>)`

使用 `IntStream`、`LongStream`、`DoubleStream` 中的 `range` 等方法没有装箱拆箱的开销。

`Optional` 对应的原始类型：`OptionalInt`、`OptionalLong`、`OptionalDouble`

## 数值范围

`range(m, n)`：生成从 m 到 n 的数值流，不含 n。
`rangeClosed(m, n)`：生成从 m 到 n 的数值流，含 n。

# 收集器

## `Collector` 接口

`Collector<T, A, R>`

- `T`：归约操作针对的元素的类型。
- `A`：容纳归约操作中间结果的累加器的类型。
- `R`：归约操作结束后的最终结果的类型。

`characteristics()` 方法

- `UNORDERED`：归约结果不受遍历和累积顺序的影响。
- `CONCURRENT`：累加器可以在多线程环境下调用和合并。
- `INDENTITY_FINISH`：不加检查直接将累加器转为最终结果，跳过完成器。

## `Collectors`

- 归约成集合容器。
  - `toCollection`
  - `toList`、`toUnmodifiableList`
  - `toSet`、`toUnmodifiableSet`
  - `toMap`、`toUnmodifiableMap`、`toConcurrentMap`
- 拼接字符串
  - `joining`
- 映射
  - `mapping`
  - `flatMapping`
- 过滤
  - `filtering`
- 统计（另有 `Long`、`Double` 版本）
  - `counting`
  - `summingInt`
  - `averagingInt`
  - `summarizingInt`
- 比较
  - `minBy`
  - `maxBy`
- 分类
  - `groupingBy`
  - `groupingByConcurrent`
  - `partitioningBy`
- 其他
  - `collectingAndThen`
  - `reducing`

# 并行流

并行流内部使用了默认的 `ForkJoinPool`，它默认的线程数量就是处理器的数量（`Runtime.getRuntime().availableProcessors()`）。

可以通过系统属性来改变线程池大小：`System.setProperty("java.util.concurrent.ForkJoinPool.common.parallelism", "4");`。

## 顺序流和并行流的转换

`Stream.parallel()`、`Stream.sequential()`

这两个方法只是修改了流内部的一个标志，流是否并行以最后一次调用的方法为准。

## 使用并行流的定性意见

- 以测试作为性能评估的最主要方式。
- 留意装箱。
- 有些操作本身就更适合顺序流，比如依赖元素顺序的操作：`limit`、`findFirst` 等。如果不一定要按顺序执行的话，可以调用 `Stream.unordered()` 方法变成无序流，再进行其他操作。
- 单个元素通过流水线的成本越高，并行流占优的可能性越高。
- 如果确定数据量较小，那么一般都不用并行流。
- 要考虑流背后的数据结构是否易于分解。比如 `ArrayList` 的拆分效率比 `LinkedList` 高得多；用 `range` 工厂方法创建的原始类型流也可以快速分解。
- 流自身的特点、以及中间操作修改流的方式，都可能改变分解过程的性能。
- 考虑终端操作中合并步骤的代价大小。

常见流数据源的可并行性

|源             |可分解性|
|:--------------|:-------|
|ArrayList      |极佳    |
|LinkedList     |差      |
|IntStream.range|极佳    |
|Stream.iterate |差      |
|HashSet        |好      |
|TreeSet        |好      |

## Spliterator

- `boolean tryAdvance(Consumer)`：按顺序依次使用 `Spliterator` 中的元素。
- `Spliterator trySplit()`：将一部分元素划分给另一个 `Spliterator`。
- `long estimateSize()`：估算剩余的元素数量。
- `int characteristics()`：特性集的编码。

`Spliterator` 的特性

|特性      |含义                                                          |
|:---------|:-------------------------------------------------------------|
|ORDERED   |元素有既定的顺序                                              |
|DISTINCT  |对于任意一对遍历过的元素 x 和 y，有 `x.equals(y) == false`    |
|SORTED    |遍历的元素按照预定义的顺序排序                                |
|SIZED     |建立自一个已知大小的源，因此 `estimate()` 返回的是准确值      |
|NONNULL   |遍历的元素不会为 null                                         |
|IMMUTABLE |数据源在遍历时不能修改                                        |
|CONCURRENT|数据源可被其他线程修改而无需同步                              |
|SUBSIZED  |当前 `Spliterator` 和它所有拆分出来的 `Spliterator` 都是 SIZED|

# 默认方法

解决继承方法冲突的规则（各条规则按优先级依次判断）

1. 优先级：类/父类 > 接口。前提是类中必须实现了该方法，而不是继承自接口的默认方法。
2. 优先选择拥有最具体实现的默认方法的接口（即子接口）。
3. 继承多个接口的类必须显式覆盖、调用期望的方法。
4. 编译错误。

# CompletableFuture

## 常用 API

工厂方法

- `supplyAsync`：传入一个 `Supplier`，以此为逻辑构造一个 `CompletableFuture`。
- `runAsync`：传入一个 `Runnable`，以此为逻辑构造一个 `CompletableFuture`。
- `completedFuture`：传入一个值，直接作为结果。

查询、获取结果

- `cancel`
- `isCancelled`
- `isCompletedExceptionally`
- `isDone`：是否完成（包括正常完成、被取消、异常）。
- `get`：获取结果；可指定超时时间；可中断。
- `join`：获取结果；**不**可指定超时时间；**不**可中断。
- `getNow`：传入一个默认值。若尚无结果，则返回默认值。
- `obtrudeValue`：强制设置结果。
- `obtrudeException`
- `allOf`：返回一个 `CompleteFuture`，当传入的若干 `CompleteFuture` 全部完成时，该 `CompleteFuture` 也完成。
- `anyOf`：返回一个 `CompleteFuture`，当传入的若干 `CompleteFuture` 任一完成时，该 `CompleteFuture` 也完成。

完成

- `complete`：传入一个值作为结果，并将 `CompletableFuture` 转入完成状态。
- `completeExceptionally`：传入一个异常作为结果，并将 `CompleteFuture` 转入完成状态。
- `completeAsync`：传入一个 `Supplier` 和 `Executor`（可选），从 `Supplier` 获取结果。
- `orTimeout`：若超时未完成，会在获取结果时抛出 `TimoutException`。
- `completeOnTimeout`：传入一个值，若超时未完成，则用该值作为结果。

函数式构造

- 每个方法分为三个版本：
  - `thenXXX`：同步版本。在当前异步线程中进行后续操作。
  - `thenXXXAsync`：异步版本。在另一个异步线程中进行后续操作。
  - `thenXXXAysnc`：异步版本。上一个方法的重载方法，传入一个 `Executor` 作为另一个异步线程的来源。
- 串行
  - 以下方法分别传入一个函数式接口（`Function`、`Consumer`、`Runnable`）。在当前 `CompleteFuture` 完成后对结果执行函数式接口，并将执行结果放在新的 `CompleteFuture` 中返回出来。
    - `thenApply`
    - `thenAccept`
    - `thenRun`
    - `thenCompose`
- 并行
  - 以下方法分别传入一个 `CompletionStage` 和一个函数式接口（`BiFunction`、`BiConsumer`、`Runnable`)。在当前和传入的 `CompletionStage` 都完成后，以两者的结果为入参执行上述函数式接口。
    - `thenCombine`
    - `thenAcceptBoth`
    - `runAfterBoth`
  - 以下方法分别传入一个 `CompletionStage` 和一个函数式接口（`Function`、`Consumer`、`Runnable`)。在当前和传入的 `CompletionStage` 任意一个完成后，以它的结果为入参执行上述函数式接口。
    - `applyToEither`
    - `acceptEither`
    - `runAfterEither`

回调

- `whenComplete`
- `handle`
- `exceptionally`


## 使用定制的执行器

`CompleteFuture` 默认使用 `ForkJoinPool.commonPool()` 作为执行器。若该线程池不支持并行，则使用 `ThreadPerTaskExecutor`（每个任务新建一个线程）。

也可以传入自己定制的 `Executor`。

**线程池的大小**

根据《Java并发编程实战》

> Nthreads = Ncpu * Ucpu * (1 + W / C)

- Nthreads：线程池中的线程数量。
- Ncpu：处理器的核心数量。通过 `Runtime.getRuntime().availableProcessors()` 得到。
- Ucpu：期望的 cpu 利用率，(0, 1]。
- W / C：等待时间/计算时间。

## 并行流和 CompletableFuture 的取舍

并行流的使用更简洁；`CompletableFuture` 对于线程池的控制更灵活。

- 如果进行的是计算密集型操作，且没有 I/O，则推荐并行流，因为这种情况下没必要创建比处理器内核更多的线程。
- 在相反的情况下，`CompletableFuture` 可以根据实际情况调整线程数。而并行流由于流的延迟特性，会很难判断到底在哪里发生了等待。


# 日期和时间 API

面向用户 `LocalDate` `LocalTime` `LocalDateTime` `ZoneDateTime`

面向机器 `Instant`

时间跨度

- `Duration`：秒、纳秒的尺度。
- `Period`：年月日的尺度。

复杂的时间操作 `TemporalAdjuster`

- `dayOfWeekInMonth`：创建一个新日期，值为同一个月中的每一周的第几天。
- `firstDayOfMonth`：创建一个新日期，值为当月的第一天。
- `firstDayOfNextMonth`：创建一个新日期，值为下月的第一天。
- `firstDayOfNextYear`：创建一个新日期，值为明年的第一天。
- `firstDayOfYear`：创建一个新日期，值为当年的第一天。
- `firstInMonth`：创建一个新日期，值为同一个月中，第一个符合星期几要求的值。
- `lastDayOfMonth`：创建一个新日期，值为当月的最后一天。
- `lastDayOfNextMonth`：创建一个新日期，值为下月的最后一天。
- `lastDayOfNextYear`：创建一个新日期，值为明年的最后一天。
- `lastDayOfYear`：创建一个新日期，值为当年的最后一天。
- `lastInMonth`：创建一个新日期，值为同一个月中，最后一个符合星期几要求的值。
- `next`/`previous`：创建一个新日期，值为之前/之后第一个符合星期几要求的日期。
- `nextOrSame`/`previousOrSame`：创建一个新日期，值为之前/之后第一个符合星期几要求的日期。若当前日期符合条件，则直接返回当前日期。

格式化

- `DateTimeFormatter`
- `DateTimeFormatterBuilder`

时区

- `ZoneId`
- `ZoneOffset`


# 函数式编程相关

## 高阶函数

- 接受至少一个函数作为参数
- 返回一个函数

## 科里化

> f(x, y) = (g(x))(y)

# 注解

## 重复注解

Java 8 的重复注解实际上是一个语法糖

```java
//首先声明一个注解容器
@interface Authors {
  Author[] value();
}

//然后声明需要重复的注解
@Repeatable(Authors.class) // 容纳重复注解的容器注解
@interface Author {
  String name();
}

//最后使用注解
@Author(name = "tom")
@Author(name = "ben")
public class Book {
  //...
}
```

`Class` 类新增了一个方法用来获取重复注解：

```java
Class#getAnnotationsByType(Class<?> clazz);
```

## 类型注解

Java 8 开始，注解可以应用于任何类型，包括 `new`、类型转换、`instance of`、泛型类型参数、`implements`、`throws`。

Java 8 之前，类型推断依赖于从上下文推断出来的目标类型。从 Java 8 开始，目标类型包括向方法传递的参数。

```java
//假设有以下方法
void doWithList(List<String> list) {
  //...
}

//之前的写法
doWithList(Collections.<String>emptyList());

//之后的写法
doWithList(Collections.emptyList());
```

# 类库的更新

## 集合

新增的方法：

- `Map`：
  - `getOrDefault`、`forEach`、`merge`、`putIfAbsent`、`remove(key, value)`、`replace`、`replaceAll`
  - `compute`：根据原有的 key-val 计算新的 val，根据新的 val 是否为 null 来删除、新增/替换 entry。返回新 val（若新 val 非 null）或 null。
  - `computeIfAbsent`：若原 val 为 null，则根据 key 计算新 val。若新 val 非 null，则 put。返回新 val（若发生了 put）或原 val。
  - `computeIfPresent`：若原 val 非 null，则根据 key 计算新 val。根据新 val 是否为 null 来删除、替换 entry。返回新 val（若发生了替换）或 null。
- `Iterable`：`forEach`、`spliterator`
- `Iterator`：`forEachRemaining`
- `Collection`：`removeIf`、`stream`、`parallelStream`
- `List`：`replaceAll`、`sort`
- `BitSet`：`stream`

## 并发

### 数值操作

Atomic 相关数字类型新增的方法：

- `getAndUpdate`：基于当前值用给定方法计算新值并更新，返回变更之前的值。
- `updateAndGet`：基于当前值用给定方法计算新值并更新，返回变更之后的值。
- `getAndAccumulate`：基于当前值和传入值用给定方法计算新值并更新，返回变更之前的值。
- `accumulateAndGet`：基于当前值和传入值用给定方法计算新值并更新，返回变更之后的值。

如果多个线程需要频繁更新且很少读取，推荐使用 `XXXAdder`、`XXXAcumulator`。

### ConcurrentHashMap

在 Java8 中，当 bucket 过于臃肿时，会动态的转换为排序树。这种转换仅当键值可比较时才会发生。

新增 3 种操作：

- `forEach`
- `reduce`
- `search`：对每个 entry 执行指定函数，直到该函数返回非空值。

每种操作支持 4 种形式：

- 使用键和值：`forEach`、`reduce`、`search`。
- 使用键：`forEachKey`、`reduceKeys`、`searchKeys`。
- 使用值：`forEachValue`、`reduceValues`、`searchValues`。
- 使用 `Map.Entry`：`forEachEntry`、`reduceEntries`、`searchEntries`。

这些操作不会对 ConcurrentHashMap 上锁。

使用时需要指定一个并发阈值。当 map 的大小小于该阈值时，操作会顺序执行。传入 1 开启基于通用线程池的最大并行。传入 Long.MAX_VALUE 以单线程执行。

新增了 `mappingCount` 方法用于统计映射的数目，它返回 long，而原有的 `size` 方法返回 int。

新增了 `KeySet` 方法返回 ConcurrentHashMap 的一个视图。

### Arrays

新增 4 个方法：

- `parallelSort`：并发排序。
- `setAll`、`parallelSetAll`：基于元素的索引计算元素值并设置。
- `parallelPrefix`：通过并发的方式，用传入的方法对数组每个元素进行累积。

### Number 和 Math

`Number` 新增方法：

- 静态方法：`sum`、`min`、`max`。
- `Integer`、`Long`：处理无符号数：`compareUnsigned`、`divideUnsigned`、`remainderUnsigned`、`toUnsignedLong`。
- `Integer`、`Long`：将字符解析为无符号数：`parseUnsignedInt`、`parseUnsignedLong`。
- `Byte`、`Short`、`Integer`：将原值视为无符号数进行转换：`toUnsignedInt`、`toUnsignedLong`。
- `Double`、`Float`：检查参数是否为有限浮点数。
- `Boolean`：执行 and、or 和 xor 操作：`logicalAnd`、`logicalOr`、`logicalXor`。
- `BigInteger`：将 `BigInteger` 转换为基础类型，若发生信息丢失会抛出算术异常：`byteValueExact`、`shortValueExact`、`intValueExact`、`longValueExact`。

`Math` 新增方法：

- 若操作发生溢出会抛出算术异常：`addExact`、`subtractExact`、`multipleExact`、`incrementExact`、`decrementExact`、`negateExact`、`toIntExact`。
- `floorMod`、`floorDiv`、`nextDown`。

## Files

现在可以用文件直接产生流：

- `Files.list`：生成指定目录中所有条目构成的 `Stream<Path>`，非递归。
- `Files.walk`：同上，不过是递归的，可设定递归深度；遍历依照深度优先。
- `Files.find`：递归遍历一个目录找到符合条件的条目，生成一个 `Stream<Path>` 对象。

## Reflection

除了为支撑注解机制变化而做的改变之外，还新增了可以查询方法参数的 API。比如可以使用 `Parameter` 类查询方法参数的名称和修饰符。

## String

新增方法 `join`，用指定分隔符拼接字符串。

# Lambda 表达式和 JVM 字节码

## 匿名类

- 编译器会为每个匿名类生成一个 .class 文件。每个 .class 文件在使用前都要加载和验证，会影响启动性能。
- 每个匿名类会为类或接口产生一个新的子类型，让 JVM 运行时性能调优更困难。

## 字节码

查看类文件的字节码和常量池：`javap -c -v {ClassName}`

lambda 表达式会被编译成 `invokedynamic` 指令。

> invokedynamic 指令（*引自《Java8 实战》附录 D.3*）
字节码指令 `invokedynamic` 最初被 jdk7 引入，用于支持运行于 JVM 上的动态类型语言。执行方法调用时，`invokedynamic` 提供了更高层的抽象，使得一部分逻辑可以依据动态语言的特征来决定调用目标。
JVM 首次执行 `invokedynamic` 时会查询一个 `bootstrap` 方法，该方法实现了依赖语言的逻辑，可以决定选择哪一个进行调用。`bootstrap` 方法返回一个链接调用点（linked call site）。当前后两次调用的是同一个方法时，不必重新选择调用的方法。调用点自身包含了一定的逻辑，可以判断何时进行重新链接。

`invokedynamic` 命令将 lambda 转换字节码的动作推迟到了运行时。这种设计的优点：

- 源代码到字节码的转换由高层的策略变成了纯粹的实现细节。它现在可以动态地改变，保留了未来优化的余地，并保持了字节码的后向兼容性。
- 没有额外的开销，没有额外的字段，不需要进行静态初始化。
- 对无状态非捕获型 lambda，可以创建 lambda 对象进行缓存。
- 没有额外的性能开销，仅在 lambda 首次被调用时需要转换。之后的调用都能直接使用之前链接的实现。

