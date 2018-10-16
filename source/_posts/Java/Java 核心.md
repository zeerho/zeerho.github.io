---
title: Java 核心
date: 2018-02-25 09:00:00
tags: [Java]
---

# 对象

基本类型直接存储值，并置于堆栈中。

|基本类型|大小|最小值   |最大值        |包装器类型|
|:-------|:---|:--------|:-------------|:---------|
|boolean |--  |--       |--            |Boolean   |
|char    |16  |Unicode 0|Unicode 2^16-1|Character |
|byte    |8   |-128     |+127          |Byte      |
|short   |16  |-2^15    |+2^15-1       |Short     |
|int     |32  |-2^31    |+2^31-1       |Integer   |
|long    |64  |-2^63    |+2^63-1       |Long      |
|float   |32  |IEEE 754 |IEEE 754      |Float     |
|double  |64  |IEEE 754 |IEEE 754      |Double    |
|void    |--  |--       |--            |Void      |

# 初始化与清理

## 成员初始化

类的成员若未在定义的时候初始化，则 Java 会给其赋默认值。而局部变量必须显式赋予初始值。

## 初始化顺序

1. 当类的静态方法/静态域首次被访问时（构造方法也是静态方法），类加载器会查找类路径，定位 xx.class 文件。
2. 载入 xx.class 文件，创建一个 Class 对象。执行所有有关静态初始化的动作（包括静态子句）。
3. 当用 new 创建对象时，现在堆上为对象分配足够的空间。
4. 这块空间会被清零，也就是将对象的成员赋予了默认初始值。
5. 执行所有出现于字段定义处的初始化动作，以及非静态子句。
6. 执行构造方法。

# Paths

# JNI 字段描述符

JNI（JavaNative Interface FieldDescriptors) 字段描述符是一种对函数返回值和参数的编码。

Boolean：Z
Byte：B
Char：C
Short：S
Int：I
Long：J
Float：F
Double：D
Void：V
对象：以"L"开头，以";"结尾，中间是用"/" 隔开的包及类名。比如：“Ljava/lang/String;”。如果是嵌套类，则用“$”来表示嵌套。例如 "(Ljava/lang/String;Landroid/os/FileUtils$FileStatus;)Z"。

# fork-join

## fork-join 最佳实践

- 调用 `join` 方法会阻塞调用方，务必在两个子任务都开始后再调用。
- 不应该在 `RecursiveTask` 内部调用 `ForkJoinPool.invoke()` 方法，而是应该调用 `compute` 或 `fork` 方法。只有顺序代码才应该调用 `invoke` 来启动并行计算。
- 只对一个子任务调用 `fork` 方法，让另一个子任务在当前线程执行。
- 如果条件允许的话，最好将任务划分为大量小任务而不是少量大任务，因为 fork-join 框架会使用“工作窃取”机制来实现负载均衡。

//TODO

# @SuppressWarnings

|警告取值类型            |用途                                                   |
|:-----------------------|:------------------------------------------------------|
|all                     |all warnings                                           |
|boxing                  |boxing/unboxing operations                             |
|cast                    |cast operations                                        |
|dep-ann                 |deprecated annotation                                  |
|deprecation             |deprecation                                            |
|fallthrough             |missing breaks in switch statements                    |
|finally                 |finally block that don’t return                        |
|hiding                  |locals that hide variable                              |
|incomplete-switch       |missing entries in a switch statement (enum case)      |
|nls                     |non-nls(National Language Support) string literals     |
|null                    |null analysis                                          |
|rawtypes                |un-specific types when using generics on class params  |
|restriction             |usage of discouraged or forbidden references           |
|serial                  |missing serialVersionUID field for a serializable class|
|static-access           |incorrect static access                                |
|synthetic-access        |unoptimized access from inner classes                  |
|unchecked               |unchecked operations                                   |
|unqualified-field-access|field access unqualified                               |
|unused                  |unused code                                            |

# javap

|option                   |meaning                                                        |
|:------------------------|:--------------------------------------------------------------|
|-c                       |输出类中各方法的未解析的代码，即构成java字节码的指令           |
|-classpath {pathlist}    |指定javap用来查找类的路径。目录用：分隔                        |
|-extdirs {dirs}          |覆盖搜索安装方式扩展的位置，扩展的缺省位置为jre/lib/ext        |
|-help                    |输出帮助信息                                                   |
|-J{flag}                 |直接将flag传给运行时系统                                       |
|-l                       |输出行及局部变量表                                             |
|-public                  |只显示public类及成员                                           |
|-protected               |只显示protected和public类及成员。                              |
|-package                 |只显示包、protected和public类及成员，这是缺省设置              |
|-private                 |显示所有的类和成员                                             |
|-s                       |输出内部类型签名                                               |
|-bootclasspath {pathlist}|指定加载自举类所用的路径，如jre/lib/rt.jar或i18n.jar           |
|-verbose                 |打印堆栈大小、各方法的locals及args参数，以及class文件的编译版本|
