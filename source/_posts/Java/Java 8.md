---
title: Java 8
date: 2017-01-01 09:00:00
tags: [Java]
---

# Lamda

## 将 lamda 表达式替换为方法引用

*参考并摘自 [developerWorks-传递表达式（pass-through lambdas）的替代方案](https://www.ibm.com/developerworks/cn/java/j-java8idioms5/index.html)。*

### 使用方法引用增加代码可读性

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
