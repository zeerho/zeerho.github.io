---
title: Groovy
date: 2018-08-07 12:00:00
tags: [Groovy]
---

# maven 引入

```xml
<dependency>
  <groupId>org.codehaus.groovy</groupId>
  <artifactId>groovy-all</artifactId>
  <version>${groovy.version}</version>
</dependency>
```

<!-- more -->

# 基础工具

## 编译

groovyc

# 数值和表达式

## 数值

- 整数：都是 `Integer` 的实例。
- 小数：都是 `BigDecimal` 的实例。

## 表达式

除法的结果包括整数和小数部分（如果有的话）。结果即使是整数也会用浮点数表示。要获得结果的整数部分需要用 `intdiv()` 方法。

对浮点数或一个含有浮点数参数的整数取模都是非法的。

## 赋值

标识符必须由字母和数字组成，大小写敏感，首字母必须是字母；下划线允许出现，以字母看待；不能是关键字。

## 关系运算符和等于运算符

比较运算符都是通过 `compareTo()` 来实现的。其中 `<=>` 运算符直接返回 `compareTo()` 的结果（-1、0、+1）。

等于/不等运算符都是通过 `equals()` 来实现的。

# 字符串和正则表达式

## 字符串字面值

```groovy
''              // 空字符串
'say "hello"!'  // 嵌套
"say 'hello'!"  // 嵌套
"""hello!"""    // 三引号单行
"""hello!       // 三引号多行
hello again!"""
```

单引号中的字符串是字面值，双引号中的 `${}` 中的内容会被求值然后作为字符串的内容。

## 字符串索引和索引段

```groovy
def s = 'abcd';
s[2]       // c
s[-1]      // d
s[1..2]    // bc
s[1..<3]   // bc
s[3..1]    // dcb
s[3, 1, 2] // dbc
```

## 基本操作

```groovy
def greeting = 'Hello world'
'Hello' + ' world'       // Hello world
'Hello' * 3              // HelloHelloHello
greeting - ' world'      // Hello
greeting.size()          // 11
greeting.length()        // 11
greeting.count('o')      // 3
greeting.contains('ell') // true
```

## 字符串方法

在 jdk 的 `String` 基础上扩展的方法。

- center：在本字符串两侧填充空格，填充后的长度由入参指定。有重载方法可以额外指定用来填充的字符串。
- eachMatch：指定一个正则表达式和一个闭包。本字符串的正则匹配结果会作为数组传入闭包。
- getAt：字符串的下标运算。
- leftShift：重载 leftShift 操作符，提供一种将多个字符串相加的更简单的方法。
- minus：删除字符串中的指定字符串。
- next：被 `String` 类的 `++` 操作符调用，用来增加字符串的末位字符。
- padLeft/padRight：左侧/右侧填充空格/指定字符串。
- plus：字符串相加。
- previous：被 `String` 类的 `--` 操作符调用，用来删除字符串的末位字符。
- reverse：逆序字符串。
- size：字符串长度。
- toCharacter/toDouble/toFloat/toInteger/toLong/toList
- tokenize：用空格/指定字符串分隔成列表。

## 正则表达式

```groovy
def reg = ~'abc' // 定义正则表达式

// 在 if 或 while 中
'abcd' =~ 'abc'  // true
'abcd' =~ reg    // true
'abcd' ==~ 'abc' // false

// 不在 if 和 while 中
'abcd' =~ 'abc'  // java.util.regex.Matcher

// 免去反斜杠的烦恼
def matcher1 = '\\abcd' =~ '\\\\(.*)'
def matcher2 = '\\abcd' =~ /\\(.*)/
```

# 列表、映射和范围

## 列表

列表是有序的。

```groovy
def arr1 = [1, 2, 3] // 定义列表
def arr2 = [1, [2, 3], 4] // 嵌套列表
def arr3 = ['abc', 1, 2] // 不同类型的元素

def ele1 = arr1[0] // 实际上是调用了 List#getAt 方法
def ele2 = [1, 2][0] // 直接量也可以索引
def ele3 = [1, 2][-1] // 反向索引
def ele4 = [1, 2][0..1] // 使用范围进行索引

arr1[1] = 4 // 赋值操作实际是调用了 List#putAt 方法
arr1 << 5 // 插入到列表末尾，实际是调用了 List#leftShift 方法
arr1 + [6, 7] // 在列表末尾拼接列表，实际是调用了 List#plus 方法
arr1 - [7] // 从列表中删除元素，实际是调用了 List#minus 方法
```

## 列表方法

GDK 新增的方法：

- flatten：使元素形式一致，返回一个新列表。
- getAt：可以指定一个索引/索引集合/范围。
- intersect：求交集。
- leftShift：重载左移运算符，在列表末尾追加元素。
- minus/plus
- pop：删除最后一个元素。
- putAt
- reverse：反转。
- sort：排序。

## 映射

```groovy
def m = ['a': 1, 'b': 2]
m['a']
m.a
```

## 映射方法

GDK 新增的方法：

- get：增加了一个重载方法，可以传入缺省值。
- getAt
- putAt

## 范围

```groovy
1..3     // 1 2 3
1..<3    // 1 2
'A'..'D' // A B C D
3..1     // 3 2 1
'C'..'A' // C B A

def start = 1
def end = 2
start..end + 1 // 1 3
```

- contains：范围中是否包含指定元素。
- get：获取范围中给定位置的元素。
- getFrom/getTo：获取最小值/最大值。
- isReverse：是否逆序。
- size
- subList

# 基本输入和输出

## 输出

print、println、printf

## 输入

见 `Console` 类。

# 方法

groovy 具有 MOP 特性：元对象协议（Meta Object Protocol）。

具体来说就是 groovy 允许动态地进行方法调用和属性访问。

## groovy 的方法调用方式

1. 除了类里本身的方法外，编译器会在编译期生成一些方法。
  1. 对于属性，会生成 getter、setter 方法。
  2. 实现 `GroovyObject` 接口的方法。
  3. 为了实现方法调用在运行期动态绑定（而不是 Java 的编译期静态绑定），生成 `$createCallSiteArray()`、`$getCallSiteArray()` 等方法。
2. 对于绝大部分方法的调用都会被编译器转换为通过 `CallSite` 进行动态调用。
  1. `CallSite` 接口定义了很多类 `call()` 方法。
  2. 很多 `CallSite` 的实现类在实例化的时候注入了 `MetaClass` 的实例，然后把方法调用委托给 `MetaClass` 实例。
3. `MetaClass` 接口类似 `Class` 类，保存了类的元信息，并提供了很多相关方法。
  - `MetaClass` 有好几个实现类，其中最重要的是 `MetaClassImpl`。
    1. 在调用方法前会做很多判断，比如是不是闭包调用、闭包调用策略、闭包的 owner 和 delegate 等等。
    2. 根据传入的调用目标的 `Class`，从 `MetaClassRegistry` 中获取对应的 `MetaClass`。
      - 这里的获取动作实际上是取 `Class#classValueMap` 字段，从中取 `ClassInfo` 对象，从中取 `Cached
    3. 把方法调用再委托给实际的 `MetaClass`。

## groovy 的方法动态绑定

groovy 对很多 Java 类进行了扩展，这些扩展的方法实际是存在于几个“类 DGM”类里（详见 `DefaultGroovyMethods#DGM_LIKE_CLASSES`，这些类以下统称“DGM”）。

groovy 将扩展方法“添加”到 Java 类的方式是：将 Java 类和扩展方法的对应关系注册到 `MetaClassRegistry` 中，这样在上一小节所述的动态调用时就能找到这些扩展方法。

`MetaClassRegistry` 的初始化发生在 `GroovySystem` 类第一次被加载时（在静态块里），也可以由用户代码主动初始化（不知道还有没有其他触发点）。

1. 在 groovy 项目本身的编译阶段：
  1. 使用 `DgmConverter` 为 DGM 中的每个公共静态方法生成一个类，名如 org.codehaus.groovy.runtime.dgm$1。注意，因为发生在编译期，所以生成的是 .class 文件，而不是 .java 文件。
  2. 将上述每个类的信息写入到 /META-INF/dgminfo 文件中。
2. 在用户项目中触发了 `MetaClassRegistry` 初始化的时候：
  1. 加载 dgminfo 文件中的所有信息。
  2. 创建所有 `CachedClass` 对象，每个对象都包含了它对应的 `Class` 和 `ClassInfo` 等信息。
  3. 为每个方法（即每个 dgm$1 类）生成一个 `Proxy` 对象，其中存放了类信息（比如类名、方法名、对应的 `CachedClass` 等等）。`Proxy` 继承自 `MetaMethod`。当第一次被调用 `invoke()` 时会加载并实例化上述的 dgm$1 类。（之所以加了这层代理是因为 dgm$1 类太多了，要把类加载和实例化推迟到真正被调用的时候。）
  4. 构建“类-方法列表”（`CachedClass`-`List<MetaMethod>`）的关联关系（实际上是把方法列表保存到了 `CachedClass` 中的 `ClassInfo` 里）。


## 默认参数

默认参数只能出现在参数列表的最后。

```groovy
def f(p1, p2 = 2, p3 = 3) {
  //...
}
```

## 作用域

方法不能引用当前方法外的变量。

# 流程控制

## for

element in Range, element in Collection, char in String, Entry in Map

## switch

```groovy
def var = 1;
switch(var) {
  case 1:
    //statement
    break
  case 2:
    //statement
    break
  default:
    //statement
    break
}
```

`case` 后面的表达式结果可以是整数、整数范围、字符串、列表、正则表达式或某些类的对象。

# 闭包

## 闭包

```groovy
//闭包是 {@code groovy.lang.Closure} 的实例
//若没有参数，可以省略形参和 ->
def greeting = "Hello"
def clo = { param1, param2 -> 
  print 'example'
  //对外部变量的引用是在定义时决定的，而不是运行时
  print '${greeting}!${param1}, ${param2}'
}

def test(clos) {
  //两种调用方式
  clo.call('a', 'b')
  clo('c', 'd')
}

//传统的调用方式
test(clo)
//简洁的调用方式（若闭包在实参列表最后，则可以放到括号后面；若实参列表只有闭包，则连括号也可以省略）
//test() clo //错误！
test() { /*blabla*/ }
test clo
test { /*blabla*/ }
```
  

