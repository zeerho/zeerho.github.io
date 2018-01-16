---
title: XML
date: 2017-01-01 09:00:00
tags: [XML]
---

# 定义

XML
> 可扩展标记语言（eXtensible Markup Language）

XML 元素
> XML 元素指的是从（且包括）开始标签直到（且包括）结束标签的部分。
一个元素可以包含：其他元素、文本、属性，或混合以上所有...

# 语法

- 声明部分 `<?xml version="1.0" encoding="utf-8" standalone="yes"?>` 可选，如果存在须放第一行；
    - `standalone="yes"` 表示该 XML 文档独立，不能引用外部的 DTD 规范文件，反之则非独立，可以引用外部规范文件。
- XML 标签
    - 所有元素都必须有关闭标签；
    - XML 标签对大小写敏感；
    - 所有标签都必须正确嵌套；
- XML 元素
    - 必须包含**根元素**，它是所有其他元素的父元素；
    - XML 元素中 `<` 和 `&` 是非法的，必须用实体引用来替代：
    |预定义实体引用|对应符号|含义|
    |:--|:--|:--|
    |`&lt;`|`<`|less than|
    |`&gt;`|`>`|greater than|
    |`&amp;`|`&`|ampersand|
    |`&apos;`|`'`|apostrophe|
    |`&quot;`|`"`|quotation mark|

    - XML 元素必须遵循以下命名规则：
        - 名称可以包含字母、数字以及其他的字符；
        - 名称不能以数字或者标点符号开始；
        - 名称不能以字母 xml（或者 XML、Xml 等等）开始；
        - 名称不能包含空格；
        - 可使用任何名称，没有保留的字词。
- XML 属性
    - 属性值必须加引号，单双引号皆可。若属性值中本身就含有单引号或双引号，则包裹属性值的引号必须用另一种，或将属性值中的引号用实体引用 `&apos;` 和 `&quot;` 替代；
    - 属性不能包含多个值（元素可以）；
    - 属性不能包含树结构（元素可以）；
    - 属性不容易扩展；
- 元数据应当存储为属性，数据本身应当存储为元素
- 注释 `<!-- This is a comment -->`;
- XML 会保留连续的多个空格，而不会裁剪为一个；
- XML 以 `LF` 存储换行。

# 格式化

1. CSS
    ```
    <?xml version="1.0" encoding="ISO-8859-1"?>
    <?xml-stylesheet type="text/css" href="cd_catalog.css"?>
    <root>
    ...
    </root>
    ```
第二行的声明把 XML 文件链接到 CSS 文件。

2. XSLT //TODO
> eXtensible Stylesheet Language Transformations
XSLT 是首选的 XML 样式表语言，它在浏览器显示 XML 文件之前，先把它转换为 HTML。

# 命名空间

命名空间是在元素的开始标签的 xmlns 属性中定义的。
命名空间声明的语法：`xmlns:前缀="URI"`。
可以为元素定义默认的命名空间来省去在所有子元素中使用前缀的工作：`xmlns="URI`。

# CDATA

1. CDATA - （未解析）字符数据 character data
术语 CDATA 是不应该由 XML 解析器解析的文本数据。
像 "<" 和 "&" 字符在 XML 元素中都是非法的。"<" 会产生错误，因为解析器会把该字符解释为新元素的开始。"&" 会产生错误，因为解析器会把该字符解释为字符实体的开始。
某些文本，比如 JavaScript 代码，包含大量 "<" 或 "&" 字符。为了避免错误，可以将脚本代码定义为 CDATA。
CDATA 部分中的所有内容都会被解析器忽略，而不会去解析其中的标签和实体引用，从而被当作普通文本来处理。
CDATA 部分由 `<![CDATA[` 开始，由 `]]>` 结束。
CDATA 部分不能包含字符串 `]]>`。也不允许嵌套的 CDATA 部分。
标记 CDATA 部分结尾的 `]]>` 不能包含空格或换行。
2. PCDATA - 被解析的字符数据 parsed character data
XML 解析器通常会解析 XML 文档中所有的文本。PCDATA 是相对于 CDATA 而言的。

# 编码

在载入一个 XML 文档时可能得到两个不同的错误，表示编码问题：

1. 在文本内容中发现无效字符。
如果 XML 中包含非 ASCII 字符，且文件保存为没有指定编码的单字节 ANSI（或 ASCII），就会得到该错误。
2. 将当前编码切换为不被支持的指定编码
如果 XML 文件保存为带有指定的单字节编码（WINDOWS-1252、ISO-8859-1、UTF-8）的双字节 Unicode（或 UTF-16），就会得到该错误。
如果 XML 文件保存为带有指定的双字节编码（UTF-16）的单字节 ANSI（或 ASCII），也会得到该错误。

结论：

- 始终使用编码属性
- 使用支持编码的编辑器
- 确保知道编辑器使用什么编码
- 在编码属性中使用相同的编码

# XML 相关技术

XHTML (可扩展 HTML) 
> 更严格更纯净的基于 XML 的 HTML 版本。

XML DOM (XML 文档对象模型)
> 访问和操作 XML 的标准文档模型。

XSL (可扩展样式表语言)，包含三个部分：
> 1. XSLT (XSL 转换) - 把 XML 转换为其他格式，比如 HTML
2. XSL-FO (XSL 格式化对象)- 用于格式化 XML 文档的语言
3. XPath - 用于导航 XML 文档的语言

XQuery (XML 查询语言)
> 基于 XML 的用于查询 XML 数据的语言。

<span id="DTD_title"></span>
[DTD](#DTD_content) (文档类型定义)
> 用于定义 XML 文档中的合法元素的标准。

<span id="XSD_title"></span>
[XSD](#XSD_content) (XML 架构)
> 基于 XML 的 DTD 替代物。

XLink (XML 链接语言)
> 在 XML 文档中创建超级链接的语言。

XPointer (XML 指针语言)
> 允许 XLink 超级链接指向 XML 文档中更多具体的部分。

SOAP (简单对象访问协议)
> 允许应用程序在 HTTP 之上交换信息的基于 XML 的协议。

WSDL (Web 服务描述语言)
> 用于描述网络服务的基于 XML 的语言。

RDF (资源描述框架)
> 用于描述网络资源的基于 XML 的语言。

RSS (真正简易聚合)
> 聚合新闻以及类新闻站点内容的格式。

SVG (可伸缩矢量图形) 
> 定义 XML 格式的图形

---
<span id="DTD_content"></span>
# DTD
[back](#DTD_title)

## 介绍
> DTD（文档类型定义）的作用是定义 XML 文档的合法构建模块。
DTD 可被成行地声明于 XML 文档中，也可作为一个外部引用。

**内部的 DOCTYPE 声明**
```xml
<?xml version="1.0"?>
<!DOCTYPE note [
<!ELEMENT note (to,from,heading,body)>
<!ELEMENT to (#PCDATA)>
<!ELEMENT from (#PCDATA)>
<!ELEMENT heading (#PCDATA)>
<!ELEMENT body (#PCDATA)>
]>
<note>
    <to>Tove</to>
    <from>Jani</from>
    <heading>Reminder</heading>
    <body>Don't forget me this weekend</body>
</note>
```

- `!DOCTYPE note` (第二行)定义此文档是 note 类型的文档。
- `!ELEMENT note` (第三行)定义 note  元素有四个元素："to、from、heading,、body"
- `!ELEMENT to` (第四行)定义 to 元素为 "#PCDATA" 类型
- `!ELEMENT from` (第五行)定义 from 元素为 "#PCDATA" 类型
- `!ELEMENT heading` (第六行)定义 heading 元素为 "#PCDATA" 类型
- `!ELEMENT body` (第七行)定义 body 元素为 "#PCDATA" 类型

**外部文档声明**
```
<?xml version="1.0"?>
<!DOCTYPE note SYSTEM "note.dtd">
<note>
    <to>Tove</to>
    <from>Jani</from>
    <heading>Reminder</heading>
    <body>Don't forget me this weekend!</body>
</note>
```

**为什么使用 DTD**

- 通过 DTD，每一个 XML 文件均可携带一个有关其自身格式的描述。
- 通过 DTD，独立的团体可一致地使用某个标准的 DTD 来交换数据。
- 应用程序可使用某个标准的 DTD 来验证从外部接收到的数据。
- 可以使用 DTD 来验证自身的数据。

---
## 元素
在 DTD 中，XML 元素通过元素声明来进行声明：
`<!ELEMENT element-name category>`
或
`<!ELEMENT element-name (element-content)>`

1. 空元素
空元素通过类别关键词 `EMPTY` 来声明
`<!ELEMENT element-name EMPTY>`
2. 只有 PCDATA 的元素
只有 PCDATA 的元素通过圆括号中的 `#PCDATA` 进行声明
`<!ELEMENT element-name (#PCDATA)>`
3. 带有任何内容的元素
通过类别关键词 `ANY` 声明的元素，可包含任何可解析数据的组合
`<!ELEMENT element-name ANY>`
4. 带有子元素（序列）的元素
带有一个或多个子元素的元素通过圆括号中的子元素名进行声明
`<!ELEMENT element-name (child1)>`
或
`<!ELEMENT element-name (child1,child2,...)>`
当子元素按照由逗号分隔开的序列进行声明时，这些子元素必须按照相同的顺序出现在文档中。在一个完整的声明中，子元素也必须被声明，同时子元素也可拥有子元素。
5. 声明必须出现且只出现一次的元素
`<!ELEMENT element-name (child-name)>`
6. 声明最少出现一次的元素
`<!ELEMENT element-name (child-name+)>`
7. 声明出现零次或多次的元素
`<!ELEMENT element-name (child-name*)>`
8. 声明出现零次或一次的元素
`<!ELEMENT element-name (child-name?)>`
9. 声明"非...即..."类型的内容
实例: `<!ELEMENT note (to,from,header,(message|body))>`
上面的例子声明了："note" 元素必须包含 "to" 元素、"from" 元素、"header" 元素，以及非 "message" 元素既 "body" 元素。
10. 声明混合型的内容
实例: `<!ELEMENT note (#PCDATA|to|from|header|message)*>`
上面的例子声明了："note" 元素可包含出现零次或多次的 PCDATA、"to"、"from"、"header" 或者 "message"。

---
## 属性
在 DTD 中，属性通过 ATTLIST 声明来进行声明：
`<!ATTLIST element-name attribute-name attribute-type attribute-value>`
DTD 实例：
`<!ATTLIST payment type CDATA "check">`
XML 实例：
`<payment type="check" />`

属性类型(attribute-type)的选项：
|类型|描述|
|:--|:--|
|CDATA|值为字符数据 (character data)|
|(en1\|en2\|..)|此值是枚举列表中的一个值|
|ID|值为唯一的 id|
|IDREF|值为另外一个元素的 id|
|IDREFS|值为其他 id 的列表|
|NMTOKEN|值为合法的 XML 名称|
|NMTOKENS|值为合法的 XML 名称的列表|
|ENTITY|值是一个实体|
|ENTITIES|值是一个实体列表|
|NOTATION|此值是符号的名称|
|xml:|值是一个预定义的 XML 值|

默认属性值可使用下列值 :
|值|解释|
|:--|:--|
|值|属性的默认值|
|#REQUIRED|属性值是必需的|
|#IMPLIED|属性不是必需的|
|#FIXED value|属性值是固定的|


---
## 实体
实体是用于定义引用普通文本或特殊字符的快捷方式的变量。

- 实体引用是对实体的引用。
- 实体可在内部或外部进行声明

一个内部实体声明：
`<!ENTITY entity-name "entity-value">`
DTD 实例:
`<!ENTITY writer "Donald Duck.">`
`<!ENTITY copyright "Copyright W3CSchool.cc">`
XML 实例：
`<author>&writer;&copyright;</author>`

一个外部实体声明：
`<!ENTITY entity-name SYSTEM "URI/URL">`
DTD 实例：
`<!ENTITY writer SYSTEM "http://www.w3cschool.cc/entities.dtd">`
`<!ENTITY copyright SYSTEM "http://www.w3cschool.cc/entities.dtd">`
XML 实例：
`<author>&writer;&copyright;</author>`

[back](#DTD_title)

---
<span id="XSD_content"></span>
# XSD
[back](#XSD_title)

> XML Schema 的作用是定义 XML 文档的合法构建模块，类似 DTD。
> 
> - 定义可出现在文档中的元素
> - 定义可出现在文档中的属性
> - 定义哪个元素是子元素
> - 定义子元素的次序
> - 定义子元素的数目
> - 定义元素是否为空，或者是否可包含文本
> - 定义元素和属性的数据类型
> - 定义元素和属性的默认值以及固定值

## < schema> 元素
**`<schema>` 元素是每一个 XML Schema 的根元素**
```
<?xml version="1.0"?>
<xs:schema xmlns:xs="http://www.w3.org/2001/XMLSchema"
targetNamespace="http://www.runoob.com"
xmlns="http://www.runoob.com"
elementFormDefault="qualified">
...
</xs:schema>
```

- `xmlns:xs="http://www.w3.org/2001/XMLSchema"`
显示 schema 中用到的元素和数据类型来自命名空间 "http://www.w3.org/2001/XMLSchema"。同时它还规定了来自命名空间 "http://www.w3.org/2001/XMLSchema" 的元素和数据类型应该使用前缀 xs
- `targetNamespace="http://www.runoob.com"`
显示被此 schema 定义的元素 (note, to, from, heading, body) 来自命名空间： "http://www.runoob.com"
- `xmlns="http://www.runoob.com"`
指出默认的命名空间是 "http://www.runoob.com"
- `elementFormDefault="qualified"`
指出任何 XML 实例文档所使用的且在此 schema 中声明过的元素必须被命名空间限定

**在 XML 文档中引用 Schema**
```
<?xml version="1.0"?>
<note xmlns="http://www.runoob.com"
xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
xsi:schemaLocation="http://www.runoob.com note.xsd">
    <to>Tove</to>
    <from>Jani</from>
    <heading>Reminder</heading>
    <body>Don't forget me this weekend!</body>
</note>
```

- `xmlns="http://www.runoob.com"`
规定了默认命名空间的声明。此声明会告知 schema 验证器，在此 XML 文档中使用的所有元素都被声明于 "http://www.runoob.com" 这个命名空间。
- 一旦拥有了可用的 XML Schema 实例命名空间：
`xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"`
就可以使用 schemaLocation 属性了。此属性有两个值。第一个值是需要使用的命名空间。第二个值是供命名空间使用的 XML schema 的位置：
`xsi:schemaLocation="http://www.runoob.com note.xsd"`

---
## 数据类型

**字符串数据类型**

- 字符串 String Data Type

> 字符串数据类型可包含字符、换行、回车以及制表符。
```
<xs:element name="customer" type="xs:string"/>
```

- 规格化字符串 NormalizedString Data Type

> 规格化字符串数据类型同样可包含字符，但是 XML 处理器会移除折行，回车以及制表符。
```
<xs:element name="customer" type="xs:normalizedString"/>
```

- Token 数据类型 Token Data Type

> Token 数据类型同样可包含字符，但是 XML 处理器会移除换行符、回车、制表符、开头和结尾的空格以及（连续的）空格。
```
<xs:element name="customer" type="xs:token"/>
```

- 字符串数据类型及衍生类型
|名称|描述|
|:--|:--|
|ENTITIES| 
|ENTITY| 
|ID|在 XML 中提交 ID 属性的字符串（仅与 schema 属性一同使用）|
|IDREF|在 XML 中提交 IDREF 属性的字符串（仅与 schema 属性一同使用）|
|IDREFS language|包含合法的语言 id 的字符串|
|Name|包含合法 XML 名称的字符串|
|NCName| 
|NMTOKEN|在 XML 中提交 NMTOKEN 属性的字符串（仅与 schema 属性一同使用）|
|NMTOKENS| 
|normalizedString|不包含换行符、回车或制表符的字符串|
|QName| 
|string|字符串|
|token|不包含换行符、回车或制表符、开头或结尾空格或者多个连续空格的字符串|


- 对字符串数据类型的限定（Restriction）
可与字符串数据类型一同使用的限定：
    - enumeration
    - length
    - maxLength
    - minLength
    - pattern (NMTOKENS、IDREFS 以及 ENTITIES 无法使用此约束)
    - whiteSpace

**日期和时间数据类型**

- 日期 Date Data Type："YYYY-MM-DD"
`<xs:element name="start" type="xs:date"/>`
    - 时区
        - 在日期后加一个 "Z" 的方式，使用世界调整时间（UTC time）来输入一个日期
    `<start>2002-09-24Z</start>`
        - 在日期后添加一个正的或负时间的方法，来规定以世界调整时间为准的偏移量
        `<start>2002-09-24-06:00</start>`
- 时间 Time Data Type："hh:mm:ss"
`<xs:element name="start" type="xs:time"/>`
    - 时区：同上
- 日期时间 DateTime Data Type："YYYY-MM-DDThh:mm:ss"，其中 T 表示必需的时间部分的起始
`<xs:element name="startdate" type="xs:dateTime"/>`
    - 时区：同上
- 持续时间 Duration Data Type："PnYnMnDTnHnMnS"，其中
    - P 表示周期(必需)
    - nY 表示年数
    - nM 表示月数
    - nD 表示天数
    - T 表示时间部分的起始 -     （如果打算规定小时、分钟和秒，则此选项为必需）
    - nH 表示小时数
    - nM 表示分钟数
    - nS 表示秒数
`<xs:element name="period" type="xs:duration"/>`
例：`<period>P5Y2M10DT15H</period>`
- 负的持续时间：在 P 前加减号
- 日期和时间数据类型
|名称|描述|
|:--|:--|
|date|定义一个日期值|
|dateTime|定义一个日期和时间值|
|duration|定义一个时间间隔|
|gDay|定义日期的一个部分 - 天 (DD)|
|gMonth|定义日期的一个部分 - 月 (MM)|
|gMonthDay|定义日期的一个部分 - 月和天 (MM-DD)|
|gYear|定义日期的一个部分 - 年 (YYYY)|
|gYearMonth|定义日期的一个部分 - 年和月 (YYYY-MM)|
|time|定义一个时间值|


- 对日期数据类型的限定（Restriction）
可与日期数据类型一同使用的限定：
    - enumeration
    - maxExclusive
    - maxInclusive
    - minExclusive
    - minInclusive
    - pattern
    - whiteSpace

**数值数据类型**

- 十进制：最大位数是 18 位
`<xs:element name="prize" type="xs:decimal"/>`
- 整数
`<xs:element name="prize" type="xs:integer"/>`
- 数值数据类型
|名字|秒数|
|:--|:--|
|byte|有正负的 8 位整数|
|decimal|十进制数|
|int|有正负的 32 位整数|
|integer|整数值|
|long|有正负的 64 位整数|
|negativeInteger|仅包含负值的整数 ( .., -2, -1.)|
|nonNegativeInteger|仅包含非负值的整数 (0, 1, 2, ..)|
|nonPositiveInteger|仅包含非正值的整数 (.., -2, -1, 0)|
|positiveInteger|仅包含正值的整数 (1, 2, ..)|
|short|有正负的 16 位整数|
|unsignedLong|无正负的 64 位整数|
|unsignedInt|无正负的 32 位整数|
|unsignedShort|无正负的 16 位整数|
|unsignedByte|无正负的 8 位整数|


- 对数值数据类型的限定（Restriction）
可与数值数据类型一同使用的限定：
    - enumeration
    - fractionDigits
    - maxExclusive
    - maxInclusive
    - minExclusive
    - minInclusive
    - pattern
    - totalDigits
    - whiteSpace

**杂项数据类型**
> 其他杂项数据类型包括布尔、base64Binary、十六进制、浮点、双精度、anyURI、anyURI 以及 NOTATION。

- 布尔 Boolean Data Type： 合法的布尔值是 true、false、1（表示 true） 以及 0（表示 false）
`<xs:attribute name="disabled" type="xs:boolean"/>`
- 二进制 Binary Data Types：
`<xs:element name="blobsrc" type="xs:hexBinary"/>`
    - 可使用两种二进制数据类型：
        - base64Binary (Base64 编码的二进制数据)
        - hexBinary (十六进制编码的二进制数据)
- AnyURI 数据类型 AnyURI Data Type：用于规定 URI；如果某个 URI 含有空格，请用 %20 替换它们
`<xs:attribute name="src" type="xs:anyURI"/>`
- 杂项数据类型
|名称|描述|
|:--|:--|
|anyURI| 
|base64Binary| 
|boolean| 
|double| 
|float| 
|hexBinary| 
|NOTATION| 
|QName|


- 对杂项数据类型的限定（Restriction）
可与杂项数据类型一同使用的限定：
    - enumeration (布尔数据类型无法使用此约束)
    - length (布尔数据类型无法使用此约束)
    - maxLength (布尔数据类型无法使用此约束)
    - minLength (布尔数据类型无法使用此约束)
    - pattern
    - whiteSpace
    
---
## 简易元素
> 简易元素指那些仅包含文本的元素。它不会包含任何其他的元素或属性。
不过，“仅包含文本”这个限定却很容易造成误解。文本有很多类型。它可以是 XML Schema 定义中包括的类型中的一种（布尔、字符串、数据等等），或者它也可以是自行定义的定制类型。
也可向数据类型添加限定（即 facets），以此来限制它的内容，或者可以要求数据匹配某种特定的模式。

**定义简易元素**
`<xs:element name="xxx" type="yyy"/>`
最常用的类型是：

- xs:string
- xs:decimal
- xs:integer
- xs:boolean
- xs:date
- xs:time

**简易元素的默认值和固定值**
简易元素可拥有指定的默认值或固定值。
当没有其他的值被规定时，默认值就会自动分配给元素：
`<xs:element name="color" type="xs:string" default="red"/>`
固定值同样会自动分配给元素，并且无法规定另外一个值：
`<xs:element name="color" type="xs:string" fixed="red"/>`

---
## 属性
> 所有的属性均作为简易类型来声明。简易元素无法拥有属性。假如某个元素拥有属性，它就会被当作某种复合类型。但是属性本身总是作为简易类型被声明的。

**定义属性**
`<xs:attribute name="xxx" type="yyy"/>`
最常用的类型是：

- xs:string
- xs:decimal
- xs:integer
- xs:boolean
- xs:date
- xs:time

**属性的默认值和固定值**
属性可拥有指定的默认值或固定值。
当没有其他的值被规定时，默认值就会自动分配给元素：
`<xs:attribute name="lang" type="xs:string" default="EN"/>`
固定值同样会自动分配给元素，并且无法规定另外的值：
`<xs:attribute name="lang" type="xs:string" fixed="EN"/>`

**可选的和必需的属性**
在缺省的情况下，属性是可选的。如需规定属性为必选，须使用 "use" 属性：
`<xs:attribute name="lang" type="xs:string" use="required"/>`

---
## 限定 / Facets
> 限定（restriction）用于为 XML 元素或者属性定义可接受的值。对 XML 元素的限定被称为 facet。

**对值的限定**
下面的例子定义了带有一个限定且名为 "age" 的元素。age 的值不能低于 0 或者高于 120：
```
<xs:element name="age">
  <xs:simpleType>
    <xs:restriction base="xs:integer">
      <xs:minInclusive value="0"/>
      <xs:maxInclusive value="120"/>
    </xs:restriction>
  </xs:simpleType>
</xs:element>
```

**对一组值的限定**
如需把 XML 元素的内容限制为一组可接受的值，我们要使用枚举约束（enumeration constraint）。
```
<xs:element name="car">
  <xs:simpleType>
    <xs:restriction base="xs:string">
      <xs:enumeration value="Audi"/>
      <xs:enumeration value="Golf"/>
      <xs:enumeration value="BMW"/>
    </xs:restriction>
  </xs:simpleType>
</xs:element>
```
上面的例子也可以被写为：
```
<xs:element name="car" type="carType"/>
<xs:simpleType name="carType">
  <xs:restriction base="xs:string">
    <xs:enumeration value="Audi"/>
    <xs:enumeration value="Golf"/>
    <xs:enumeration value="BMW"/>
  </xs:restriction>
</xs:simpleType>
```
注意： 在这种情况下，类型 "carType" 可被其他元素使用，因为它不是 "car" 元素的组成部分。

**对一系列值的限定**
如需把 XML 元素的内容限制定义为一系列可使用的数字或字母，我们要使用模式约束（pattern constraint）。
```
<xs:element name="letter">
    <xs:simpleType>
        <xs:restriction base="xs:string">
            <!-- 实际应用中取下列任一元素 -->
            <!-- 只接受小写字母 a - z 其中的一个 -->
            <xs:pattern value="[a-z]"/>
            <!-- 只接受大写字母 A - Z 其中的三个 -->
            <xs:pattern value="[A-Z][A-Z][A-Z]"/>
            <!-- 只接受 a - z 中零个或多个字母 -->
            <xs:pattern value="([a-z])*"/>
            <!-- 只接受一对或多对字母，每对字母由一个小写字母后跟一个大写字母组成 -->
            <xs:pattern value="([a-z][A-Z])+"/>
            <!-- 只接受 male 或者 female -->
            <xs:pattern value="male|female"/>
            <!-- 只接受由 8 个字符组成的一行字符，这些字符必须是大写或小写字母 a - z 亦或数字 0 - 9 -->
            <xs:pattern value="[a-zA-Z0-9]{8}"/>
        </xs:restriction>
    </xs:simpleType>
</xs:element>
```

**对空白字符的限定**
如需规定对空白字符（whitespace characters）的处理方式，需要使用 whiteSpace 限定。
下面的例子定义了带有一个限定的名为 "address" 的元素。这个 whiteSpace 限定被设置为 "preserve"，这意味着 XML 处理器不会移除任何空白字符：
```
<xs:element name="address">
    <xs:simpleType>
        <xs:restriction base="xs:string">
            <!-- 实际应用中取下列任一元素 -->
            <!-- 不会移除任何空白字符 -->
            <xs:whiteSpace value="preserve"/>
            <!-- 将移除所有空白字符（换行、回车、空格以及制表符） -->
            <xs:whiteSpace value="replace"/>
            <!-- 将移除所有空白字符（换行、回车、空格以及制表符会被替换为空格，开头和结尾的空格会被移除，而多个连续的空格会被缩减为一个单一的空格） -->
            <xs:whiteSpace value="collapse"/>
        </xs:restriction>
    </xs:simpleType>
</xs:element>
```

**对长度的限定**
如需限制元素中值的长度，需要使用 length、maxLength 以及 minLength 限定。
本例定义了带有一个限定且名为 "password" 的元素。其值必须精确到 8 个字符：
```
<xs:element name="password">
    <xs:simpleType>
        <xs:restriction base="xs:string">
            <!-- 值必须精确到 8 个字符 -->
            <xs:length value="8"/>
            <!-- 值最小为 5 个字符，最大为 8 个字符 -->
            <xs:minLength value="5"/>
            <xs:maxLength value="8"/>
        </xs:restriction>
    </xs:simpleType>
</xs:element>
```

**数据类型的限定**
|限定|描述|
|:--|:--|
|enumeration|定义可接受值的一个列表|
|fractionDigits|定义所允许的最大的小数位数。必须大于等于0。|
|length|定义所允许的字符或者列表项目的精确数目。必须大于或等于0。|
|maxExclusive|定义数值的上限。所允许的值必须小于此值。|
|maxInclusive|定义数值的上限。所允许的值必须小于或等于此值。|
|maxLength|定义所允许的字符或者列表项目的最大数目。必须大于或等于0。|
|minExclusive|定义数值的下限。所允许的值必需大于此值。|
|minInclusive|定义数值的下限。所允许的值必需大于或等于此值。|
|minLength|定义所允许的字符或者列表项目的最小数目。必须大于或等于0。|
|pattern|定义可接受的字符的精确序列。|
|totalDigits|定义所允许的阿拉伯数字的精确位数。必须大于0。|
|whiteSpace|定义空白字符（换行、回车、空格以及制表符）的处理方式。|


---
## 复合元素
> 复合元素指包含其他元素及/或属性的 XML 元素。
> 
> - 空元素
> - 包含其他元素的元素
> - 仅包含文本的元素
> - 包含元素和文本的元素
> 
> 上述元素均可包含属性。

**定义复合元素**
这个复合 XML 元素，"employee"，仅包含其他元素：
```
<employee>
    <firstname>John</firstname>
    <lastname>Smith</lastname>
</employee>
```
在 XML Schema 中，有两种方式来定义复合元素：

1. 通过命名此元素，可直接对 "employee" 元素进行声明：
```
<xs:element name="employee">
  <xs:complexType>
    <xs:sequence>
      <xs:element name="firstname" type="xs:string"/>
      <xs:element name="lastname" type="xs:string"/>
    </xs:sequence>
  </xs:complexType>
</xs:element>
```
假如使用上面所描述的方法，那么仅有 "employee" 可使用所规定的复合类型。请注意其子元素，"firstname" 以及 "lastname"，被包围在指示器 `<sequence>` 中。这意味着子元素必须以它们被声明的次序出现。
2. "employee" 元素可以使用 type 属性，这个属性的作用是引用要使用的复合类型的名称：
```
<xs:element name="employee" type="personinfo"/>

<xs:complexType name="personinfo">
  <xs:sequence>
    <xs:element name="firstname" type="xs:string"/>
    <xs:element name="lastname" type="xs:string"/>
  </xs:sequence>
</xs:complexType>
```
如果使用了上面所描述的方法，那么若干元素均可以使用相同的复合类型：
```
<xs:element name="employee" type="personinfo"/>
<xs:element name="student" type="personinfo"/>
<xs:element name="member" type="personinfo"/>

<xs:complexType name="personinfo">
  <xs:sequence>
    <xs:element name="firstname" type="xs:string"/>
    <xs:element name="lastname" type="xs:string"/>
  </xs:sequence>
</xs:complexType>
```
也可以在已有的复合元素之上以某个复合元素为基础，然后添加一些元素：
```
<xs:element name="employee" type="fullpersoninfo"/>

<xs:complexType name="personinfo">
  <xs:sequence>
    <xs:element name="firstname" type="xs:string"/>
    <xs:element name="lastname" type="xs:string"/>
  </xs:sequence>
</xs:complexType>

<xs:complexType name="fullpersoninfo">
  <xs:complexContent>
    <xs:extension base="personinfo">
      <xs:sequence>
        <xs:element name="address" type="xs:string"/>
        <xs:element name="city" type="xs:string"/>
        <xs:element name="country" type="xs:string"/>
      </xs:sequence>
    </xs:extension>
  </xs:complexContent>
</xs:complexType>
```

### **复合类型-空元素**
> 空的复合元素不能包含内容，只能含有属性。

一个空的 XML 元素：
```
<product prodid="1345" />
```
上面的 "product" 元素根本没有内容。为了定义无内容的类型，就必须声明一个在其内容中只能包含元素的类型，但是实际上并不会声明任何元素，比如这样：
```
<xs:element name="product">
  <xs:complexType>
        <xs:complexContent>
            <xs:restriction base="xs:integer">
                <xs:attribute name="prodid" type="xs:positiveInteger"/>
            </xs:restriction>
        </xs:complexContent>
    </xs:complexType>
</xs:element>
```
在上面的例子中，定义了一个带有复合内容的复合类型。complexContent 元素给出的信号是，打算限定或者拓展某个复合类型的内容模型，而 integer 限定则声明了一个属性但不会引入任何的元素内容。
但是，也可以更加紧凑地声明此 "product" 元素：
```
<xs:element name="product">
  <xs:complexType>
    <xs:attribute name="prodid" type="xs:positiveInteger"/>
  </xs:complexType>
</xs:element>
```
或者可以为一个 complexType 元素起一个名字，然后为 "product" 元素设置一个 type 属性并引用这个 complexType 名称（通过使用此方法，若干个元素均可引用相同的复合类型）：
```
<xs:element name="product" type="prodtype"/>

<xs:complexType name="prodtype">
  <xs:attribute name="prodid" type="xs:positiveInteger"/>
</xs:complexType>
```

### **复合类型-仅含元素**
> "仅含元素"的复合类型元素是只能包含其他元素的元素。

```
<person>
    <firstname>John</firstname>
    <lastname>Smith</lastname>
</person>
```
可在 schema 中这样定义 "person" 元素：
```
<xs:element name="person">
  <xs:complexType>
    <xs:sequence>
      <xs:element name="firstname" type="xs:string"/>
      <xs:element name="lastname" type="xs:string"/>
    </xs:sequence>
  </xs:complexType>
</xs:element>
```
注意，它意味着被定义的元素必须按上面的次序出现在 "person" 元素中。
或者可以为 complexType 元素设定一个名称，并让 "person" 元素的 type 属性来引用此名称（如使用此方法，若干元素均可引用相同的复合类型）：
```
<xs:element name="person" type="persontype"/>

<xs:complexType name="persontype">
  <xs:sequence>
    <xs:element name="firstname" type="xs:string"/>
    <xs:element name="lastname" type="xs:string"/>
  </xs:sequence>
</xs:complexType>
```

### **复合类型-仅含文本**
> 含文本的复合元素可包含文本和属性。

此类型仅包含简易的内容（文本和属性），因此要向此内容添加 simpleContent 元素。当使用简易内容时，我们就必须在 simpleContent 元素内定义扩展或限定：
```
<xs:element name="somename">
  <xs:complexType>
    <xs:simpleContent>
      <xs:extension base="basetype">
        ....
        ....
      </xs:extension>
    </xs:simpleContent>
  </xs:complexType>
</xs:element>
```
或者：
```
<xs:element name="somename">
  <xs:complexType>
    <xs:simpleContent>
      <xs:restriction base="basetype">
        ....
        ....
      </xs:restriction>
    </xs:simpleContent>
  </xs:complexType>
</xs:element>
```
提示： 请使用 extension 或 restriction 元素来扩展或限制元素的基本简易类型。 这里有一个 XML 元素的例子，"shoesize"，其中仅包含文本：
```
<shoesize country="france">35</shoesize>
```
下面这个例子声明了一个复合类型，其内容被定义为整数值，并且 "shoesize" 元素含有名为 "country" 的属性：
```
<xs:element name="shoesize">
  <xs:complexType>
    <xs:simpleContent>
      <xs:extension base="xs:integer">
        <xs:attribute name="country" type="xs:string" />
      </xs:extension>
    </xs:simpleContent>
  </xs:complexType>
</xs:element>
```
也可为 complexType 元素设定一个名称，并让 "shoesize" 元素的 type 属性来引用此名称（通过使用此方法，若干元素均可引用相同的复合类型）：
```
<xs:element name="shoesize" type="shoetype"/>

<xs:complexType name="shoetype">
  <xs:simpleContent>
    <xs:extension base="xs:integer">
      <xs:attribute name="country" type="xs:string" />
    </xs:extension>
  </xs:simpleContent>
</xs:complexType>
```

### **复合类型-混合内容**
> 混合的复合类型可包含属性、元素以及文本。

```
<letter>
    Dear Mr.<name>John Smith</name>.
    Your order <orderid>1032</orderid>
    will be shipped on <shipdate>2001-07-13</shipdate>.
</letter>
```
下面这个 schema 声明了这个 "letter" 元素：
```
<xs:element name="letter">
  <xs:complexType mixed="true">
    <xs:sequence>
      <xs:element name="name" type="xs:string"/>
      <xs:element name="orderid" type="xs:positiveInteger"/>
      <xs:element name="shipdate" type="xs:date"/>
    </xs:sequence>
  </xs:complexType>
</xs:element>
```
注意： 为了使字符数据可以出现在 "letter" 的子元素之间，mixed 属性必须被设置为 "true"。`<xs:sequence>` 标签 (name、orderid 以及 shipdate ) 意味着被定义的元素必须依次出现在 "letter" 元素内部。
也可以为 complexType 元素起一个名字，并让 "letter" 元素的 type 属性引用 complexType 的这个名称（通过这个方法，若干元素均可引用同一个复合类型）：
```
<xs:element name="letter" type="lettertype"/>

<xs:complexType name="lettertype" mixed="true">
  <xs:sequence>
    <xs:element name="name" type="xs:string"/>
    <xs:element name="orderid" type="xs:positiveInteger"/>
    <xs:element name="shipdate" type="xs:date"/>
  </xs:sequence>
</xs:complexType>
```

---
## 指示器
> 通过指示器，可以控制在文档中使用元素的方式。
> 
> - Order 指示器：
>   - All
>   - Choice
>   - Sequence
> - Occurrence 指示器：
>   - maxOccurs
>   - minOccurs
> - Group 指示器：
>   - Group name
>   - attributeGroup name

**Order 指示器**
> Order 指示器用于定义元素的顺序。

- All 指示器
规定子元素可以按照任意顺序出现，且每个子元素必须只出现一次：
```
<xs:element name="person">
  <xs:complexType>
    <xs:all>
      <xs:element name="firstname" type="xs:string"/>
      <xs:element name="lastname" type="xs:string"/>
    </xs:all>
  </xs:complexType>
</xs:element>
```
注意： 当使用 all 指示器时，可以把 `<minOccurs>` 设置为 0 或者 1，而只能把 `<maxOccurs>` 指示器设置为 1。

- Choice 指示器
规定可出现某个子元素或者可出现另外一个子元素（非此即彼）：
```
<xs:element name="person">
  <xs:complexType>
    <xs:choice>
      <xs:element name="employee" type="employee"/>
      <xs:element name="member" type="member"/>
    </xs:choice>
  </xs:complexType>
</xs:element>
```

- Sequence 指示器
规定子元素必须按照特定的顺序出现：
```
<xs:element name="person">
   <xs:complexType>
    <xs:sequence>
      <xs:element name="firstname" type="xs:string"/>
      <xs:element name="lastname" type="xs:string"/>
    </xs:sequence>
  </xs:complexType>
</xs:element>
```

**Occurrence 指示器**
> 用于定义某个元素出现的频率。

注意： 对于所有的 "Order" 和 "Group" 指示器（any、all、choice、sequence、group name 以及 group reference），其中的 maxOccurs 以及 minOccurs 的默认值均为 1。

- maxOccurs 指示器
可规定某个元素可出现的最大次数：
```
<xs:element name="person">
  <xs:complexType>
    <xs:sequence>
      <xs:element name="full_name" type="xs:string"/>
      <xs:element name="child_name" type="xs:string" maxOccurs="10"/>
    </xs:sequence>
  </xs:complexType>
</xs:element>
```
上面的例子表明，子元素 "child_name" 可在 "person" 元素中最少出现一次（其中 minOccurs 的默认值是 1），最多出现 10 次。

- minOccurs 指示器
可规定某个元素能够出现的最小次数：
```
<xs:element name="person">
  <xs:complexType>
    <xs:sequence>
      <xs:element name="full_name" type="xs:string"/>
      <xs:element name="child_name" type="xs:string"
      maxOccurs="10" minOccurs="0"/>
    </xs:sequence>
  </xs:complexType>
</xs:element>
```
上面的例子表明，子元素 "child_name" 可在 "person" 元素中出现最少 0 次，最多出现 10 次。
如需使某个元素的出现次数不受限制，请使用 maxOccurs="unbounded" 这个声明：
一个实际的例子：

**Group 指示器**
> 用于定义相关的数批元素。

- 元素组
通过 group 声明进行定义：
```
<xs:group name="groupname">
...
</xs:group>
```
必须在 group 声明内部定义一个 all、choice 或者 sequence 元素。下面这个例子定义了名为 "persongroup" 的 group，它定义了必须按照精确的顺序出现的一组元素：
```
<xs:group name="persongroup">
  <xs:sequence>
    <xs:element name="firstname" type="xs:string"/>
    <xs:element name="lastname" type="xs:string"/>
    <xs:element name="birthday" type="xs:date"/>
  </xs:sequence>
</xs:group>
```
在把 group 定义完毕以后，就可以在另一个定义中引用它了：
```
<xs:group name="persongroup">
  <xs:sequence>
    <xs:element name="firstname" type="xs:string"/>
    <xs:element name="lastname" type="xs:string"/>
    <xs:element name="birthday" type="xs:date"/>
  </xs:sequence>
</xs:group>

<xs:element name="person" type="personinfo"/>

<xs:complexType name="personinfo">
  <xs:sequence>
    <xs:group ref="persongroup"/>
    <xs:element name="country" type="xs:string"/>
  </xs:sequence>
</xs:complexType>
```

- 属性组
通过 attributeGroup 声明来进行定义：
```
<xs:attributeGroup name="groupname">
...
</xs:attributeGroup>
```
下面这个例子定义了名为 "personattrgroup" 的一个属性组：
```
<xs:attributeGroup name="personattrgroup">
  <xs:attribute name="firstname" type="xs:string"/>
  <xs:attribute name="lastname" type="xs:string"/>
  <xs:attribute name="birthday" type="xs:date"/>
</xs:attributeGroup>
```
在已定义完毕属性组之后，就可以在另一个定义中引用它了：
```
<xs:attributeGroup name="personattrgroup">
  <xs:attribute name="firstname" type="xs:string"/>
  <xs:attribute name="lastname" type="xs:string"/>
  <xs:attribute name="birthday" type="xs:date"/>
</xs:attributeGroup>

<xs:element name="person">
  <xs:complexType>
    <xs:attributeGroup ref="personattrgroup"/>
  </xs:complexType>
</xs:element>
```

---
## any 元素
> any 元素使我们有能力通过未被 schema 规定的元素来拓展 XML 文档。

下面这个例子是从名为 "family.xsd" 的 XML schema 中引用的片段。它展示了一个针对 "person" 元素的声明。通过使用 any 元素，我们可以通过任何元素（在 `<lastname>` 之后）扩展 "person" 的内容：
```
<xs:element name="person">
  <xs:complexType>
    <xs:sequence>
      <xs:element name="firstname" type="xs:string"/>
      <xs:element name="lastname" type="xs:string"/>
      <xs:any minOccurs="0"/>
    </xs:sequence>
  </xs:complexType>
</xs:element>
```
现在，我们希望使用 "children" 元素来扩展 "person" 元素。这此种情况下就可以这么做，即使以上这个 schema 的作者没有声明任何 "children" 元素。
请看这个 schema 文件，名为 "children.xsd"：
```
<?xml version="1.0" encoding="ISO-8859-1"?>
<xs:schema xmlns:xs="http://www.w3.org/2001/XMLSchema"
targetNamespace="http://www.w3schools.com"
xmlns="http://www.w3schools.com"
elementFormDefault="qualified">
    <xs:element name="children">
      <xs:complexType>
        <xs:sequence>
          <xs:element name="childname" type="xs:string"
          maxOccurs="unbounded"/>
        </xs:sequence>
      </xs:complexType>
    </xs:element>
</xs:schema>
```
下面这个 XML 文件（名为 "Myfamily.xml"），使用了来自两个不同的 schema 中的成分，"family.xsd" 和 "children.xsd"：
```
<?xml version="1.0" encoding="ISO-8859-1"?>
<persons xmlns="http://www.microsoft.com"
xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
xsi:schemaLocation="http://www.microsoft.com family.xsd
http://www.w3schools.com children.xsd">
    <person>
      <firstname>Hege</firstname>
      <lastname>Refsnes</lastname>
      <children>
        <childname>Cecilie</childname>
      </children>
    </person>
    <person>
      <firstname>Stale</firstname>
      <lastname>Refsnes</lastname>
    </person>
</persons>
```
上面这个 XML 文件是有效的，这是由于 schema "family.xsd" 允许我们通过在 "lastname" 元素后的可选元素来扩展 "person" 元素。

---
## anyAttribute
> anyAttribute 元素使我们有能力通过未被 schema 规定的属性来扩展 XML 文档。

下面的例子是来自名为 "family.xsd" 的 XML schema 的一个片段。它为我们展示了针对 "person" 元素的一个声明。通过使用 `<anyAttribute>` 元素，我们就可以向 "person" 元素添加任意数量的属性：
```
<xs:element name="person">
  <xs:complexType>
    <xs:sequence>
      <xs:element name="firstname" type="xs:string"/>
      <xs:element name="lastname" type="xs:string"/>
    </xs:sequence>
    <xs:anyAttribute/>
  </xs:complexType>
</xs:element>
```
现在，我们希望通过 "gender" 属性来扩展 "person" 元素。在这种情况下我们就可以这样做，即使这个 schema 的作者从未声明过任何 "gender" 属性。
请看这个 schema 文件，名为 "attribute.xsd"：
```
<?xml version="1.0" encoding="ISO-8859-1"?>
<xs:schema xmlns:xs="http://www.w3.org/2001/XMLSchema"
targetNamespace="http://www.w3schools.com"
xmlns="http://www.w3schools.com"
elementFormDefault="qualified">
    <xs:attribute name="gender">
      <xs:simpleType>
        <xs:restriction base="xs:string">
          <xs:pattern value="male|female"/>
        </xs:restriction>
      </xs:simpleType>
    </xs:attribute>
</xs:schema>
```
下面这个 XML（名为 "Myfamily.xml"），使用了来自不同 schema 的成分，"family.xsd" 和 "attribute.xsd"：
```
<?xml version="1.0" encoding="ISO-8859-1"?>
<persons xmlns="http://www.microsoft.com"
xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
xsi:SchemaLocation="http://www.microsoft.com family.xsd
http://www.w3schools.com attribute.xsd">
    <person gender="female">
      <firstname>Hege</firstname>
      <lastname>Refsnes</lastname>
    </person>
    <person gender="male">
      <firstname>Stale</firstname>
      <lastname>Refsnes</lastname>
    </person>
</persons>
```
上面这个 XML 文件是有效的，这是因为 schema "family.xsd" 允许我们向 "person" 元素添加属性。

---
## 元素替换
> 通过元素替换 (Element Substitution)，一个元素可对另一个元素进行替换。

**元素替换**
```
<xs:element name="name" type="xs:string"/>
<xs:element name="navn" substitutionGroup="name"/>

<xs:complexType name="custinfo">
  <xs:sequence>
    <xs:element ref="name"/>
  </xs:sequence>
</xs:complexType>

<xs:element name="customer" type="custinfo"/>
<xs:element name="kunde" substitutionGroup="customer"/>
```
有效的 XML 文档类似这样（根据上面的 schema）：
```
<customer>
  <name>John Smith</name>
</customer>
```
或类似这样：
```
<kunde>
  <navn>John Smith</navn>
</kunde>
```

**阻止元素替换**
为防止其他的元素替换某个指定的元素，须使用 block 属性：
```
<xs:element name="name" type="xs:string" block="substitution"/>
```
某个 XML schema 的片段：
```
<xs:element name="name" type="xs:string" block="substitution"/>
<xs:element name="navn" substitutionGroup="name"/>

<xs:complexType name="custinfo">
  <xs:sequence>
    <xs:element ref="name"/>
  </xs:sequence>
</xs:complexType>

<xs:element name="customer" type="custinfo" block="substitution"/>
<xs:element name="kunde" substitutionGroup="customer"/>
```
合法的 XML 文档应该类似这样（根据上面的 schema）：
```
<customer>
  <name>John Smith</name>
</customer>
```
但是下面的文档不再合法：
```
<kunde>
  <navn>John Smith</navn>
</kunde>
```

**使用 substitutionGroup**
可替换元素的类型必须和主元素相同，或者从主元素衍生而来。假如可替换元素的类型与主元素的类型相同，那么就不必规定可替换元素的类型了。
注意，substitutionGroup 中的所有元素（主元素和可替换元素）必须被声明为**全局元素**，否则就无法工作！

**什么是全局元素（Global Elements）**
全局元素指 "schema" 元素的直接子元素！本地元素（Local elements）指嵌套在其他元素中的元素。

[back](#XSD_title)
