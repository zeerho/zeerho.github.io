# Freemarker

标签： !Java

[TOC]

---

## Freemarker 模板文件的组成
- **文本：**直接输出的部分。
- **注释：**`<#-- 注释 -->`格式部分，不会输出。
- **插值：**即`${...}`或`#{...}`格式的部分，将被数据模型中的对应数据所替代并输出。
- **FTL 指令：**和 HTML 标记类似，名字前加`#`予以区分，不会输出。

下面是一个 FreeMarker 模板的例子,包含了以上所说的 4 个部分
```HTML
<html>
    <head>
        <title>Welcome!</title>
    </head>
    <body>
    <#-- 注释部分，不会输出到HTML中 -->
    <#-- 下面使用插值 -->
    <h1>Welcome ${user} !</h1>
    <p>We have these animals:<br>
    <#-- 使用FTL指令 -->
    <ul>
        <#list animals as being>
        <li>${being.name} for ${being.price} Euros</li>
        <#list>
    </ul>
    </body>
</html>
```

### 1. FTL 指令规则
在 FreeMarker 中，使用 FTL 标签来使用指令，FreeMarker 有3种 FTL 标签，这和 HTML 标签是完全类似的。

- **开始标签：** `<#directivename parameter>`
- **结束标签：** `</#directivename>`
- **空标签：** `<#directivename parameter/>`

实际上，使用标签时前面的`#`也可能变成`@`，如果该指令是一个用户指令而不是系统内建指令，应将`#`改成`@`。
使用 FTL 标签时，应该有正确的嵌套，而不是交叉使用，这和 XML 标签的用法完全一样。如果全用不存在的指令，FreeMarker 不会使用模板输出，而是产生一个错误消息。FreeMarker 会忽略 FTL 标签中的空白字符。值得注意的是`<` 、`/>` 和指令之间不允许有空白字符。

### 2. 插值规则
FreeMarker 的插值有如下两种类型:

- 通用插值：`${expr};`
- 数字格式化插值：`#{expr}`或`#{expr;format}`

#### 2.1 通用插值
对于通用插值,又可以分为以下 4 种情况:
**1. 插值结果为字符串值：**直接输出表达式结果
**2. 插值结果为数字值：**根据默认格式（由`#setting`指令设置）将表达式结果转换成文本输出.可以使用内建的字符串函数格式化单个插值,如下面的例子:
```
<#setting number_format="currency"/>
<#assign answer=42/>
${answer}
${answer?string} <#-- the same as ${answer} -->
${answer?string.number}
${answer?string.currency}
${answer?string.percent}
${answer}
```
输出结果是:
```
$42.00
$42.00
42
$42.00
4,200%
```
**3.插值结果为日期值：**根据默认格式(由`#setting`指令设置)将表达式结果转换成文本输出。可以使用内建的字符串函数格式化单个插值,如下面的例子：
```
${lastUpdated?string("yyyy-MM-dd HH:mm:ss zzzz")}
${lastUpdated?string("EEE, MMM d, ''yy")}
${lastUpdated?string("EEEE, MMMM dd, yyyy, hh:mm:ss a '('zzz')'")}
```
输出结果是：
```
2008-04-08 08:08:08 Pacific Daylight Time
Tue, Apr 8, '03
Tuesday, April 08, 2003, 08:08:08 PM (PDT)
```
**4. 插值结果为布尔值：**根据默认格式(由`#setting`指令设置)将表达式结果转换成文本输出。可以使用内建的字符串函数格式化单个插值，如下面的例子：
```
<#assign foo=true/>
${foo?string("yes", "no")}
```
输出结果是:
```
yes
```
#### 2.2 数字格式化插值
数字格式化插值可采用`#{expr;format}`形式来格式化数字,其中`format`可以是:
`mX`:小数部分最小X位
`MX`:小数部分最大X位
如下面的例子：
```
<#assign x=2.582/>
<#assign y=4/>
#{x; M2} <#-- 输出2.58 -->
#{y; M2} <#-- 输出4 -->
#{x; m2} <#-- 输出2.6 -->
#{y; m2} <#-- 输出4.0 -->
#{x; m1M2} <#-- 输出2.58 -->
#{x; m1M2} <#-- 输出4.0 -->
```


