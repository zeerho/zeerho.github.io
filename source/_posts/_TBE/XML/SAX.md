# SAX

标签： !XML

[TOC]

---

## SAX

> Simple API for XML

- 是基于事件的 API。
- 在一个比 DOM 低的级别上操作。
- 提供比 DOM 更多的控制。
- 几乎总是比 DOM 更有效率。
- 需要比 DOM 更多的工作。

---
## 常用的 SAX 接口和类

SAX 将其事件分为几个接口：

- **ContentHandler** 定义与文档本身关联的事件（例如，开始和结束标记）。大多数应用程序都注册这些事件。
- **DTDHandler** 定义与 DTD 关联的事件。然而，它不定义足够的事件来完整地报告 DTD。如果需要对 DTD 进行语法分析，请使用可选的 DeclHandler。DeclHandler 是 SAX 的扩展，并且不是所有的语法分析器都支持它。
- **EntityResolver** 定义与装入实体关联的事件。只有少数几个应用程序注册这些事件。
- **ErrorHandler** 定义错误事件。许多应用程序注册这些事件以便用它们自己的方式报错。

为简化工作，SAX 在 DefaultHandler 类中提供了这些接口的缺省实现。在大多数情况下，为应用程序扩展 DefaultHandler 并覆盖相关的方法要比直接实现一个接口更容易。

---
## XMLReader

为注册事件处理器并启动语法分析器，应用程序使用 XMLReader 接口中的相关方法，启动语法分析：
```
parser.parse(args[0]);
```

**XMLReader 的主要方法是：**

- **parse()** 对 XML 文档进行语法分析。 parse() 有两个版本；一个接受文件名或 URL，另一个接受 InputSource 对象。
- **setContentHandler() 、 setDTDHandler() 、 setEntityResolver() 和 setErrorHandler()** 让应用程序注册事件处理器。
- **setFeature() 和 setProperty()** 控制语法分析器如何工作。它们采用一个特性或功能标识（一个类似于名称空间的 URI 和值）。功能采用 Boolean 值，而特性采用“对象”。

**最常用的 XMLReaderFactory 功能是：**

- **http://xml.org/sax/features/namespaces**，所有 SAX 语法分析器都能识别它。如果将它设置为 true（缺省值），则在调用 ContentHandler 的方法时，语法分析器将识别出名称空间并解析前缀。
- **http://xml.org/sax/features/validation**，它是可选的。如果将它设置为 true，则验证语法分析器将验证该文档。非验证语法分析器忽略该功能。

---
## XMLReaderFactory

XMLReaderFactory 创建语法分析器对象。它定义 createXMLReader() 的两个版本：一个采用语法分析器的类名作为参数，另一个从 org.xml.sax.driver 系统特性中获得类名称。
对于 Xerces，类是 org.apache.xerces.parsers.SAXParser 。应该使用 XMLReaderFactory ，因为它易于切换至另一种 SAX 语法分析器。实际上，只需要更改一行然后重新编译。
```
XMLReader parser = XMLReaderFactory.createXMLReader(
                       "org.apache.xerces.parsers.SAXParser");
```
为获得更大的灵活性，应用程序可以从命令行读取类名或使用不带参数的 createXMLReader() 。因此，甚至可以不重新编译就更改语法分析器。

---
## InputSource

InputSource 控制语法分析器如何读取文件，包括 XML 文档和实体。
在大多数情况下，文档是从 URL 装入的。但是，有特殊需求的应用程序可以覆盖 InputSource 。例如，这可以用来从数据库中装入文档。

---
## ContentHandler

ContentHandler 是最常用的 SAX 接口，因为它定义 XML 文档的事件。

ContentHandler 声明下列事件：

- **startDocument() / endDocument()** 通知应用程序文档的开始或结束。
- **startElement() / endElement()** 通知应用程序标记的开始或结束。属性作为 Attributes 参数传递。即使只有一个标记，“空”元素（例如， `<img href="logo.gif"/>`）也生成 startElement() 和 endElement() 。
- **startPrefixMapping() / endPrefixMapping()** 通知应用程序命名空间作用域。通常不需要该信息，因为当 http://xml.org/sax/features/namespaces 为 true 时，语法分析器已经解析了名称空间。
- 当语法分析器在元素中发现文本（已经过语法分析的字符数据）时， **characters() / ignorableWhitespace()** 会通知应用程序。要知道，语法分析器负责将文本分配到几个事件（更好地管理其缓冲区）。 ignorableWhitespace 事件用于由 XML 标准定义的可忽略空格。
- **processingInstruction()** 将处理指令通知应用程序。
- **skippedEntity()** 通知应用程序已经跳过了一个实体（即，当语法分析器未在 DTD/schema 中发现实体声明时）。
- **setDocumentLocator()** 将 Locator 对象传递到应用程序。请注意，不需要 SAX 语法分析器提供 Locator ，但是如果它提供了，则必须在任何其它事件之前激活该事件。

---
## 属性

在 startElement() 事件中，应用程序在 Attributes 参数中接收属性列表。
```
String attribute = attributes.getValue("","price");
```
Attributes 参数仅在 startElement() 事件期间可用。如果在事件之间需要它，则用 AttributesImpl 复制一个。

Attributes 定义下列方法：

- **getValue(i) / getValue(qName) /getValue(uri,localName)** 返回第 i 个属性值或给定名称的属性值。
- **getLength()** 返回属性数目。
- **getQName(i) / getLocalName(i) /getURI(i)** 返回限定名（带前缀）、本地名（不带前缀）和第 i 个属性的名称空间 URI。
- **getType(i) / getType(qName) / getType(uri,localName)** 返回第 i 个属性的类型或者给定名称的属性类型。类型为字符串，即在 DTD 所使用的： “CDATA”、“ID”、“IDREF”、“IDREFS”、“NMTOKEN”、“NMTOKENS”、“ENTITY”、“ENTITIES” 或 “NOTATION”。

---
## 定位器

Locator 为应用程序提供行和列的位置。不需要语法分析器来提供 Locator 对象。

Locator 定义下列方法：

- **getColumnNumber()** 返回当前事件结束时所在的那一列。在 endElement() 事件中，它将返回结束标记所在的最后一列。
- **getLineNumber()** 返回当前事件结束时所在的行。在 endElement() 事件中，它将返回结束标记所在的行。
- **getPublicId()** 返回当前文档事件的公共标识。
- **getSystemId()** 返回当前文档事件的系统标识。

---
## DTDHandler

DTDHandler 声明两个与 DTD 语法分析器相关的事件。

- **notationDecl()** 通知应用程序已经声明了一个标记。
- **nparsedEntityDecl()** 通知应用程序已经发现了一个未经过语法分析的实体声明。

---
## EntityResolver

EntityResolver 接口仅定义一个事件 resolveEntity() ，它返回 InputSource。
因为 SAX 语法分析器已经可以解析大多数 URL，所以很少应用程序实现 EntityResolver 。例外情况是目录文件，它将公共标识解析成系统标识。如果在应用程序中需要目录文件，请下载 Norman Walsh 的目录软件包。

---
## ErrorHandler

ErrorHandler 接口定义错误事件。处理这些事件的应用程序可以提供定制错误处理。
安装了定制错误处理器后，语法分析器不再抛出异常。抛出异常是事件处理器的责任。
接口定义了与错误的三个级别或严重性对应的三个方法：

- **warning()** 警示那些不是由 XML 规范定义的错误。例如，当没有 XML 声明时，某些语法分析器发出警告。它不是错误（因为声明是可选的），但是它可能值得注意。
- **error()** 警示那些由 XML 规范定义的错误。
- **fatalError()** 警示那些由 XML 规范定义的致命错误。

---
## SAXException

SAX 定义的大多数方法都可以抛出 SAXException 。当对 XML 文档进行语法分析时， SAXException 通知一个错误。
错误可以是语法分析错误也可以是事件处理器中的错误。要报告来自事件处理器的其它异常，可以将异常封装在 SAXException 中。



















