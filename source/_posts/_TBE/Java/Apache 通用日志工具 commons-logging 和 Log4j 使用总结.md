---
title: Apache 通用日志工具 commons-logging 和 Log4j 使用总结
date: 2017-01-01 09:00:00
tags: [Java]
---

**Apache 为了让众多日志工具有一个相同操作方法，制作了一个通用日志工具包 commos-logging。**

# Log4j 的架构

Log4j三大板块：日志写入器、日志输出终端、日志布局模式。
![](http://img1.51cto.com/attachment/200705/200705091178696293444.png)
`Logger`类是日志包的核心，`Lgger`的名称大小写敏感，并且名称之间有继承关系——子名由父名做前缀，用点号“.”分隔。
`Logger`系统中有个根`logger`，是所有`logger`的祖先，它总是存在，且不可通过名字获取，可以通过`Logger.getRootLogger()`获取。

**`Logger` 对象获取方法，具体参考 API 文档**
`static Logger getLogger(Class clazz)`
*Shorthand for getLogger(clazz.getName()).*
`static Logger getLogger(String name)`
*Retrieve a logger named according to the value of the name parameter.*
`static Logger getLogger(String name, LoggerFactory factory)`
*Like getLogger(String) except that the type of logger instantiated depends on the type returned by the LoggerFactory.makeNewLoggerInstance(java.lang.String) method of the factory parameter.*
`static Logger getRootLogger()`
*Return the root logger for the current logger rpository.*
在某对象中，用该对象所属的类作为参数，调用`Logger.getLogger(Class clazz)`以获取`logger`对象被认为是目前所知最明智的命名`logger`方法。
用同名参数调用`Logger.getLogger(Class clazz)`将返回同一个`logger`对象。`Logger`的创建可以按照任意顺序，`log4j`将自动维护`logger`的继承树。

# Log4j的日志级别（Level）
每个`logger`都有一个日志级别，用来控制日志的输出。未分配级别的`logger`将自动继承它最近的父`logger`的日志级别。
**`Logger`的级别由低到高：**
**ALL<DEBUG<INFO<WARN<ERROR<FATAL<OFF**
*`Logger`的级别越低，输出的日志越详细。

# Log4j的输出终端（Appender接口）

`Log4j`提供了以下几个实现：

- org.apache.log4j.ConsoleAppender（控制台）
- org.apache.log4j.FileAppender（文件）
- org.apache.log4j.DailyRollingFileAppender（每天都产生一个日志文件）
- org.apache.log4j.RollingFileAppender（文件大小达到指定尺寸时产生一个新的日志文件，文件名称上会自动添加数字序号。）
- org.apache.log4j.WriterAppender（将日志信息以流的格式发送到任意指定的地方）
默认情况下，子`logger`将继承父`logger`的所有`appenders`。
`rootlogger`拥有目标为`system.out`的`consoleAppender`，故默认情况下，所有`logger`都将继承该`appender`。

# Log4j的输出布局模式（Layout接口）
`Log4j`提供Layout有以下几种：

- org.apache.log4j.HTMLLayout（以HTML表格形式布局）
- org.apache.log4j.PatternLayout（可以灵活地指定布局模式）
- org.apache.log4j.SimpleLayout（包含日志信息的级别和信息字符串）
- org.apache.log4j.TTCCLayout（包含日志产生的时间、线程、类别等信息）
`Log4j`采用类似C语言中的printf函数的打印格式格式化日志信息。参数如下：

- `%m`：输出代码中指定的消息。
- `%p`：输出优先级。
- `%r`：输入自应用启动到输出该log信息耗费的毫秒数。
- `%c`：输出所属的类目，通常就是所在类的全名。
- `%t`：输出产生该日志线程的线程名。
- `%n`：输出一个回车换行符。Windows平台为“\r\n”，UNIX平台为“\n”。
- `%d`：输出日志时间点的日期或时间，默认格式为ISO8601，推荐使用“%d{ABSOLUTE}”，这个输出格式形如“2007-05-07 18:21:13,500”，符合中国人习惯。
- `%l`：输出日志事件发生的位置，包括类名、线程名，以及所在代码的行数。

# Log4j的配置

`Log4j`一般通过配置文件配置使用。配置文件有两种：Java properties和XML。一般选用properties文件来配置，因为简洁易读。
对`Log4j`的配置就是对`rootLogger`和子`Logger`的配置。主要的配置项为：`rootLogger`、输出终端、输出布局模式。
所有的配置项须以`log4j`开头。
**log4j.properties**
```
##logger的配置###
#配置根logger
log4j.rootLogger=INFO,stdout
#配置子logger:org.lavasoft（在org.lavasoft包中类的日志在没有指定logger名的情况下使用这个logger）
log4j.logger:org.lavasoft=ERROR,file
#配置子logger:org.lavasoft.test（在org.lavasoft.test包中类的日志在没有指定logger名的情况下使用这个logger）
log4j.logger:org.lavasoft.test=ERROR,file1,stout

## direct log message to stdout ###（标准的终端输出）
#控制台输出
log4j.appender.stdout=org.apache.log4j.ConsoleAppender
#自定义输出布局
log4j.appender.stdout.layout=org.apache.log4j.PatternLayout
#输出的格式
log4j.appender.stdout.layout.ConversionPattern=%d{ABSOLUTE} %5p %c{1};%L - %m%n

## direct messages to file ttt.log ###（输入到文件ttt.log的配置）
#输出到滚动文件
log4j.appender.file1=org.apache.log4j.RollingFileAppender
#输出文件最大为10M
log4j.appender.file1.MaxFileSize=10MB
#输出文件最大序号为10
log4j.appender.file1.MaxBackupIndex=10
#输出文件路径
log4j.appender.file1.File=C:/ttt1.log
#自定义输出布局
log4j.appender.file1.layout=org.apache.log4j.PatternLayout
#输出格式
log4j.appender.file1.layout.ConversionPattern=%d %-5p [%t] (%13F:%L) %3x - %m%n
```
logger的配置语法为：级别，输入终端1，输出终端2，……
根logger的配置项为：log4j.rootLogger
子logger的配置项为：log4j.logger.<子logger名>

# Log4j与通用日志包commons-logging的结合

其实commons-logging中默认都支持Log4j，因此只要同时加载`commons-logging`包和`log4j`包，可以不用配置即可用在应用中使用`commons-logging`的接口方法。
当然，标准的应用的是需要的配置，如果你log4j则这个配置是可选的。下面我说明如何通过配置文件来组合commons-logging和log4j。
 
配置文件内容很简单，就指定一个日志实现类即可，下面是个示例文件：
 
**commons-logging.properties**
```
org.apache.commons.logging.Log=org.apache.commons.logging.impl.Log4JLogger
```
 
在使用Log4j作为日志工具的时候，commons-logging.properties的配置可以不要.
