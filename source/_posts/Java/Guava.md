---
title: Guava
date: 2019-02-02 14:20:00
tags: [Java]
---

# base

## 先决条件校验

`Preconditions`

- `checkArgument(boolean expr) throws IllegalArgumentException` 校验参数是否符合条件。
- `checkState(boolean expr) throws IllegalStateException` 校验状态。
- `checkNotNull(T ref) throws NullPointerException` 校验空指针。
- `checkElementIndex(int index, int size) throws IndexOutOfBoundsException` 校验索引越界（0 <= index < size）。
- `checkPositionIndex(int index, int size) throws IndexOutOfBoundsException` 校验位置越界（0 <= index <= size）。

注释中提到了一个有意思的问题。

在 2009 年版本的 hotspot 中，它偏爱将异常信息表达式重构为一个返回字符串的方法（如下）。

```java
if (guardExpression) {
  throw new BadException(stringException);
}
// 重构
if (guardExpression) {
  throw new BadException(badMsg(...));
}
```

但是如果重构为一个返回 `void` 或 `Exception` 的方法的话运行速度会慢很多。这是 hotspot 优化器的一个 bug。

`Preconditions` 类中的方法普遍需要“根据参数来抛出不同类型的异常”。为了避免上述性能问题，作者将参数传入 `badMsg` 方法，并在其中判断参数然后抛出不同类型的异常或者返回异常信息字符串。由于 `badMsg` 的返回类型依然是 `String`，所以不会有性能问题。这么做本质上是把生成不同类型异常的方法放到某个异常对象的构造过程当中，而 hotspot 在这种情况下没有优化问题。

不知道之后版本的 hotspot 有没有解决这个问题。

# 对象相关工具方法

## Objects

- `equal` 空指针安全的相等判断。Java 7 开始应使用 `Objects#equal`。
- `hash` 为多个对象生成一个 hashCode。Java 7 开始应使用 `Objects#hash`。

## MoreObjects

- `firstNonNull` 返回参数中第一个非空对象。
- `toStringHelper` 方便重写对象的 `toString` 方法。

```java
// Returns "ClassName{x=1}"
MoreObjects.toStringHelper(this)
  .add("x", 1)
  .toString();

// Returns "MyObject{x=1}"
MoreObjects.toStringHelper("MyObject")
  .add("x", 1)
  .toString();
```

## ComparisonChain

链式调用进行多个 compare 操作，一旦确定结果就不会进行后续比较。

```java
@Override
public int compareTo(T o) {
  return ComparisionChain.start()
      .compare(this.aString, that.aString)
      .compare(this.anInt, that.anInt)
      //...
      .result();
}
```

## Joiner

用来进行字符串的拼接。此类的实例时不可变的，因此是线程安全的。

```java
// 首先调用构建方法
Joiner joiner = Joiner.on(";") // 分隔符
    .useForNull("bla")         // 拼接过程中用来替换 null
    .skipNulls();              // 拼接过程中忽略 null，跟上个方法二选一
// 拼接 Map
Joiner mapJoiner = Joiner.on(";") // 各 k-v 之间的分隔符
    .withKeyValueSeparator("=");  // k 和 v 之间的分隔符

String result1 = joiner.join("a", null, "b"); // 进行拼接，有多个重载方法
Appendable result2 = joiner.appendTo(appendable, iterable); // 将 iterable 依次拼接入 appendable，会加入分隔符，有多个重载方法
```

## Splitter

分割字符串。可以定制一些分割条件。

