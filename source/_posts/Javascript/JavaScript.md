---
title: JavaScript
date: 2017-01-01 09:00:00
tags: [JavaScript]
---

<!-- more -->

# 词法结构

## 字符集

- 区分大小写。
- 转义序列表示任一 Unicode 内码：`\u01AB`，其中后 4 位是可变 16 进制数。注释中的转义序列不会被解释为 Unicode 内码。

## 标识符和保留字

- 标识符必须以**字符**、**下划线**或**美元符号（$）**开始，后续可以是字母、数字、下划线或美元符。
- 出于可移植性和可读性的考虑，通常只用 ASCII 字母和数字来书写标识符。但从技术上来讲，JavaScript 允许标识符中出现 Unicode 字符全集中的字母和数字。ECMAScript 标准也允许在标识符首字符后面出现 Unicode 字符集中的 Mn 类、Mc 类和 Pc 类[^1]。

**JavaScript 保留关键字**

|-           |-         |-      |-       |-         |-       |-        |
|:-----------|:---------|:------|:-------|:---------|:-------|:--------|
|abstract    |arguments |boolean|break   |byte      |case    |catch    |
|char        |class#    |const  |continue|debugger  |default |delete   |
|do          |double    |else   |enum#   |eval      |export# |extends# |
|false       |final     |finally|float   |for       |function|goto     |
|if          |implements|import#|in      |instanceof|int     |interface|
|let         |long      |native |new     |null      |package |private  |
|protected   |public    |return |short   |static    |super#  |switch   |
|synchronized|this      |throw  |throws  |transient |true    |try      |
|typeof      |var       |void   |volatile|while     |with    |yield    |

*`#` 标记的关键字是 ECMAScript5 中新添加的。*

**JavaScript 对象、属性和方法**

应该避免使用 JavaScript 内置的对象、属性和方法的名称作为 Javascript 的变量或函数名

|-        |-       |-       |-            |-             |
|:--------|:-------|:-------|:------------|:-------------|
|Array    |Date    |eval    |function     |hasOwnProperty|
|Infinity |isFinite|isNaN   |isPrototypeOf|length        |
|Math     |NaN     |name    |Number       |Object        |
|prototype|String  |toString|undefined    |valueOf       |

**Java 保留关键字**

JavaScript 经常与 Java 一起使用，应该避免使用一些 Java 对象和属性作为 JavaScript 标识符

|-       |-   |-        |-        |-         |-          |
|:-------|:---|:--------|:--------|:---------|:----------|
|getClass|java|JavaArray|javaClass|JavaObject|JavaPackage|

**Windows 保留关键字**

JavaScript 可以在 HTML 外部使用。它可在许多其他应用程序中作为编程语言使用。
在 HTML 中，必须（为了可移植性，也应该这么做）避免使用 HTML 和 Windows 对象和属性的名称作为 Javascript 的变量及函数名

|-             |-                 |-          |-                 |-                 |
|:-------------|:-----------------|:----------|:-----------------|:-----------------|
|alert         |all               |anchor     |anchors           |area              |
|assign        |blur              |button     |checkbox          |clearInterval     |
|clearTimeout  |clientInformation |close      |closed            |confirm           |
|constructor   |crypto            |decodeURI  |decodeURIComponent|defaultStatus     |
|document      |element           |elements   |embed             |embeds            |
|encodeURI     |encodeURIComponent|escape     |event             |fileUpload        |
|focus         |form              |forms      |frame             |innerHeight       |
|innerWidth    |layer             |layers     |link              |location          |
|mimeTypes     |navigate          |navigator  |frames            |frameRate         |
|hidden        |history           |image      |images            |offscreenBuffering|
|open          |opener            |option     |outerHeight       |outerWidth        |
|packages      |pageXOffset       |pageYOffset|parent            |parseFloat        |
|parseInt      |password          |pkcs11     |plugin            |prompt            |
|propertyIsEnum|radio             |reset      |screenX           |screenY           |
|scroll        |secure            |select     |self              |setInterval       |
|setTimeout    |status            |submit     |taint             |text              |
|textarea      |top               |unescape   |untaint           |window            |

**HTML 事件句柄**

应该避免使用 HTML 事件句柄的名称作为 Javascript 的变量及函数名。

|-        |-         |-          |-          |
|:--------|:---------|:----------|:----------|
|onblur   |onclick   |onerror    |onfocus    |
|onkeydown|onkeypress|onkeyup    |onmouseover|
|onload   |onmouseup |onmousedown|onsubmit   |

**非标准 JavaScript**

除了保留关键字，在 JavaScript 实现中也有一些非标准的关键字。
一个实例是 `const` 关键字，用于定义变量。 一些 js 引擎把 `const` 当作 `var` 的同义词。另一些引擎则把 `const` 当作只读变量的定义。
`const` 是 js 的扩展。js 引擎支持它用在 Firefox 和 Chrome 中。但是它并不是 JavaScript 标准 ES3 或 ES5 的组成部分。建议：不要使用它。

## 可选的分号

- js 只有在缺少了分号就无法正确解析代码时才会自动填补分号。有两个例外：
    - 若 `return`、`break` 和 `continue` 后紧跟换行，js 会填补分号；
    - 对于 `++` 和 `--` 运算符，若既可以解释为表达式前缀，又可以解释为后缀，js 会优先解释为前缀；
- 通常以 `(`、`[`、`/`、`+`、`-` 开始的语句较有可能和前一条语句一起解析，保守起见可在此类语句前加分号，这样即使前一条语句结尾的分号被误删了，当前语句也能正常执行。

# 类型、值和变量

- 基本类型：数字、字符串、布尔值、null、undefined。
- 除基本类型外都是对象类型。js 核心定义了五种类：Array、Function、Date、RegExp、Error。

## 数字

- 采用 IEEE 754 标准定义的 64 位浮点格式，最大值 ±1.7976931348623157x10^308，最小值 ±5x10^-324。整数范围 -2^53~2^53，包含边界值。若超过此范围则无法保证低位的精度。
- **Javascript 在实际操作时基于 32 位整数**，这在操作大数时可能会出现意外的情况。
- ECMAScript 标准不支持八进制数字，但某些 JavaScript 实现允许以数字 0 开始的八进制数字。在 ES6 的严格模式下，八进制直接量是明令禁止的。
- 数字表示方式：`[digits][.digits][(E|e)[+|-]digits]`。
- 数字溢出以 `Infinity` `-Infinity` 表示，基于它们的加减乘除结果还是无穷大。
- 数字下溢以 `0` `-0` 表示。
- 零除以零、无穷大除以无穷大、负数做开方、数字运算符与非数字或无法转换成数字的对象一起使用时，会返回 `NaN`。
- ECMAScript 3 中 `Infinity` 和 `NaN` 是可读可写可修改的，ES5 中修复了这个 bug。
- NaN 与任何值都不相等，包括自身。也就是说不能通过 `x==NaN` 来判断，而要通过 `x!=x` 来判断。或者使用 `isNaN()` 函数判断。
- 正负零值严格相等，除了它们作为除数时得到的结果。

## 文本

- 字符串是一组由 16 位值组成的不可变的有序序列，每个字符通常来自于 Unicode 字符集。
- js 采用 UTF-16 编码，这意味着字符串的长度是其所含 16 位值的个数，而不是字面上的字符个数。
- js 定义的字符串操作方法均作用于 16 位值，而非字符，且不会对代理项做单独处理。同样，js 不会对字符串做标准化加工，甚至不能保证字符串是合法的 UTF-16 格式。
- js 中没有表示单个字符的“字符类型”。
- 在 ES3 中，字符串直接量必须写在一行中。在 ES5 中，可以将字符串拆分成数行，每行以 `\` 结束，`\` 和后面隐含的换行符都不算字符串直接量的组成部分。如果想换行，可以使用 `\n`。
- 若 `\` 后的字符不存在对应的转义字符，则会忽略 `\`，仍解释为该字符本身。
- 在 ES5 中，字符串可以当作只读数组，使用 charAt() 方法或方括号来访问其中的字符。以 Mozilla 为首的多数浏览器都在 ES5 之前就支持这种特性，除了 IE。

|转义字符|含义                               |
|:-------|:----------------------------------|
|`\o`    |NUL 字符（\u0000）                 |
|`\b`    |退格符（\u0008）                   |
|`\t`    |水平制表符（\u0009）               |
|`\n`    |换行符（\u000A）                   |
|`\v`    |垂直制表符（\u000B）               |
|`\f`    |换页符（\u000C）                   |
|`\r`    |回车符（\u000D）                   |
|`\"`    |双引号（\u0022）                   |
|`\'`    |撇号或单引号（\u0027）             |
|`\\`    |反斜线（\u005C）                   |
|`\xXX`  |由两位十六进制数指定的 Latin-1 字符|
|`\uXXXX`|由四位十六进制数制定的 Unicode 字符|

## 布尔

任意 js 的值都可以转换为布尔值。

以下值会被转换为 false：

- undefined
- null
- 0
- -0
- NaN
- ""

其他所有值（包括对象）都会被转换为 true。

## null 和 undefined

- typeof(null) === "object"
- typeof(undefined) === "undefined"
- null == undefined --> true
- null === undefined --> false

## 全局对象

当 js 解释器启动时，或浏览器加载新页面时，将创建一个新的全局对象，并给它一组定义的初始属性：

- 全局属性，比如 undefined，Infinity，NaN
- 全局函数，比如 isNaN()，parseInt()，eval()
- 构造函数，比如 Date()，RegExp()，String()，Object()，Array()
- 全局对象，比如 Math，JSON

全局对象的初始属性不是保留字，但应该当作保留字来对待。

## 类型转换

|值                  |字符串        |数字|布尔值|对象                 |
|:-------------------|:-------------|:---|:-----|:--------------------|
|undefined           |"undefined"   |NaN |false |throws TypeError     |
|null                |"null"        |0   |false |throws TypeError     |
|                    |              |    |      |                     |
|true                |"true"        |1   |      |new Boolean(true)    |
|false               |"false"       |0   |      |new Boolean(false)   |
|                    |              |    |      |                     |
|""                  |              |0   |false |new String("")       |
|"1.2"               |              |1.2 |true  |new String("1.2")    |
|"one"               |              |NaN |true  |new String("one")    |
|                    |              |    |      |                     |
|0                   |"0"           |    |false |new Number(0)        |
|-0                  |"0"           |    |false |new Number(-0)       |
|NaN                 |"NaN"         |    |false |new Number(NaN)      |
|Infinity            |"Infinity"    |    |true  |new Number(Infinity) |
|-Infinity           |"-Infinity"   |    |true  |new Number(-Infinity)|
|1                   |"1"           |    |true  |new Number(1)        |
|                    |              |    |      |                     |
|{}任意对象          |\#1           |\#2 |true  |                     |
|[]任意数组          |""            |0   |true  |                     |
|[9]1个数字元素      |"9"           |9   |true  |                     |
|['a']其它数组       |使用join()方法|NaN |true  |                     |
|function(){}任意函数|@             |NaN |true  |                     |


1. 对象 -> 字符串：
    - 尝试 toString() 方法，若返回一个原始值，则转为字符串。
    - 若无 toString() 方法，或返回的不是原始值，则调用 valueOf() 方法，若返回一个原始值，则转为字符串。
    - 若无法通过以上两种方法获取原始值，则抛出类型异常。
2. 对象 -> 数字：
    - 尝试 valueOf() 方法，若返回一个原始值，则转为数字。
    - 否则，尝试 toString() 方法，将得到的字符串转为数字。
    - 否则，抛出类型异常。

注意：

- 一个值转换为另一个值并不意味两个值“相等”。比如在期望布尔值的地方使用了 undefined，它将会转换 false，但这并不表明 undefined == false。“==”运算符从不试图将其操作数转换为布尔值。
- 对于 Boolean()、Number()、String() 或 Object()，当不通过 new 调用这些函数时，它们会作为类型转换函数。
- 如果试图把 undefined 和 null 转换为对象，会抛出 TypeError，但用 Object() 显式转换不会抛异常，而是返回一个空对象。
- 数字转字符串的不同方法（3 个方法都会适当地四舍五入或填充 0）：
    - toFixed()：指定小数点后的位数，从不使用指数记数法。
    - toExponential()：使用指数记数法，其中小数点前固定 1 位，小数点后位数由参数决定。
    - toPrecision()：指定有效数字的位数，若指定的有效数字位数小于整数部分位数，则转换成指数形式。
- 使用 Number() 将字符串转换为数字直接量，只能基于十进制，且不能出现非法的尾随字符。
- parseInt() 只解析整数，parseFloat() 可解析整数和浮点数。两者都会跳过任意数量的前导空格，并尽可能多地解析字符，并忽略后面无法解析的部分。若第一个非空字符即非数字，则返回 NaN。若字符串前缀为 0x 或 0X，则 parseInt() 将其解释为十六进制数。parseInt() 可接受第二个参数来指定基数，范围 2~36。

# 表达式和运算符

## 运算符概述

|运算符      |操作                        |结合性|操作数|类型            |
|:-----------|:---------------------------|:-----|:-----|:---------------|
|++          |前/后增量                   |←     |1     |lval→num        |
|--          |前/后增量                   |←     |1     |lval→num        |
|-           |求反                        |←     |1     |num→num         |
|+           |转换为数字                  |←     |1     |num→num         |
|~           |按位求反                    |←     |1     |int→int         |
|!           |逻辑非                      |←     |1     |bool→bool       |
|delete      |删除属性                    |←     |1     |lval→bool       |
|typeof      |检测操作数类型              |←     |1     |any→str         |
|void        |返回 undefined 值           |←     |1     |any→undef       |
|            |                            |      |      |                |
|\*、/、%    |乘、除、取余                |→     |2     |num,num→num     |
|            |                            |      |      |                |
|+、-        |加、减                      |→     |2     |num,num→num     |
|+           |字符串拼接                  |→     |2     |str,str→str     |
|            |                            |      |      |                |
|<<          |左位移                      |→     |2     |int,int→int     |
|\>>         |有符号右移                  |→     |2     |int,int→int     |
|\>>>        |无符号右移                  |→     |2     |int,int→int     |
|            |                            |      |      |                |
|<、<=、>、>=|比较数字顺序                |→     |2     |num,num→bool    |
|<、<=、>、>=|比较在字母表中的顺序        |→     |2     |str,str→bool    |
|instanceof  |测试对象类                  |→     |2     |obj,func→bool   |
|in          |测试属性是否存在            |→     |2     |str,obj→bool    |
|            |                            |      |      |                |
|==          |判断相等                    |→     |2     |any,any→bool    |
|!=          |判断不等                    |→     |2     |any,any→bool    |
|===         |判断恒等                    |→     |2     |any,any→bool    |
|!==         |判断非恒等                  |→     |2     |any,any→bool    |
|            |                            |      |      |                |
|&           |按位与                      |→     |2     |int,int→int     |
|            |                            |      |      |                |
|^           |按位异或                    |→     |2     |int,int→int     |
|            |                            |      |      |                |
|\|          |按位或                      |→     |2     |int,int→int     |
|            |                            |      |      |                |
|&&          |逻辑与                      |→     |2     |any,any→any     |
|            |                            |      |      |                |
|\|\|        |逻辑或                      |→     |2     |any,any→any     |
|            |                            |      |      |                |
|?:          |条件运算符                  |←     |3     |bool,any,any→any|
|            |                            |      |      |                |
|=           |变量赋值或对象属性赋值      |→     |2     |lval,any→any    |
|\*=、/=、%=、+=、-=、&=、^=、\|=、<<=、>>=、>>>=|运算且赋值|→|2|lval,any→any|
|            |                            |      |      |                |
|,           |忽略第一个、返回第二个操作数|→     |2     |any,any→any     |


## 位运算符

- 位运算符要求操作数是整数，并表示为 32 位整型而不是 64 位浮点型。必要时位运算符会先将操作数转换为数字并强制表示为 32 位整型，这会忽略小数部分和超过32位的二进制部分。
- 移位运算符要求右运算符在 0~31 之间。
- 位运算符会将 NaN、Infinity 和 -Infinity 都转换为 0。

## 逻辑表达式

- 逻辑与（&&）和逻辑或（||）将所有操作数都当作真、假值来计算，并将其中某个操作数当作真、假值结果来返回。
- 考虑到逻辑表达式具有短路的性质，因此当计算到某个操作数就可以确定整个逻辑表达式的结果时，就不再继续计算，且将该操作数作为真、假值结果来返回。
- 逻辑非（!）运算符与 && 和 || 不同，它不是将操作数当作真、假值，而是将操作数转换为布尔值然后取反。

## 算数表达式

**“+”运算符**

- 若其中一个操作数为对象，则按照规则将其转换为原始类值。
- 转换之后，若其中一个操作数为字符串，则将另一个也转为字符串然后进行拼接。
- 否则，将两者都转为数字或 NaN，然后做加法。

## 其他运算符

**typeof**

|x                     |typeof x   |
|:---------------------|:----------|
|undefined             |"undefined"|
|null                  |"object"   |
|true/flase            |"boolean"  |
|任意数字或 NaN        |"number"   |
|任意字符串            |"string"   |
|任意函数              |"function" |
|任意内置对象（非函数）|"object"   |
|任意宿主对象|由编译器各自实现的字符串，但不是"undefined"、"boolean"、"number"或"string"|

# 语句

## 严格模式

严格模式是 ES5 的一个受限制的子集。与非严格模式的区别：

- 禁止使用 with 语句。
- 如果给一个未声明的变量、函数、函数参数、catch 从句参数或全局对象的属性赋值，将抛出一个引用错误异常。（在非严格模式中会给全局对象添加一个属性）
- 调用的函数（不是方法）中的 this 值是 undefined。（在非严格模式中是全局对象）。可以此判断是否支持严格模式：`var hasStrictMode = (function(){"use strict"; return this === undefined}());`
- 当通过 `call()` 或 `apply()` 来调用函数时，其中的 this 值就是 `call()` 或 `apply()` 传入的第一个参数（在非严格模式中 null 和 undefined 值被全局对象和转换为对象的非对象值代替）。
- 给只读属性赋值和给不可扩展的对象创建成员都会抛出类型错误异常（非严格模式中只是失败，不会抛异常）。
- 传入 `eval()` 的代码不能在调用程序所在的上下文中声明变量或定义函数。变量和函数的定义是在 `eval()` 创建的新作用域中，这个作用域在 `eval()` 返回时弃用。
- 函数里的 arguments 对象拥有传入函数值的静态副本。在非严格模式中，两者是指向同一个值的引用。
- 当 delete 运算符后跟随非法标识符（变量、函数、函数参数）时，将抛出语法错误异常（在非严格模式中什么也不做，并返回 false）。
- 在一个对象直接量中定义两个或多个同名属性将产生一个语法错误（非严格模式下不会报错）。
- 函数声明中存在两个或多个同名参数将产生一个语法错误（非严格模式下不会报错）。
- 不允许使用八进制整数直接量（以 0 为前缀）。
- 标识符 `eval` 和 `arguments` 当作关键字，它们的值不能修改，不能给它们赋值、声明为变量、用作函数名、用作函数参数或用作 catch 块的标识符。
- 限制了对调用栈的检测能力。在严格模式的函数中，arguments.caller 和 arguments.callee 都会抛出一个类型错误异常。严格模式的函数同样具有 caller 和 arguments 属性，访问它们时会抛出类型错误异常（一些 js 实现在非严格模式里定义了这些非标准的属性）。

# 对象

## 创建对象

### 对象直接量

- 属性名字里有空格或连字符的，必须用字符串表示。
- ES5 和部分 ES3 的实现中，保留字可以用作不带引号的属性名。保险起见，属性名字是保留字的，要用字符串表示。
- ES5 中，对象直接量中最后一个属性后的逗号会被忽略。在 ES3 的大部分实现中也会忽略，除了 IE。

### Object.create()

ES5 中定义了 Object.create() 方法，第一个参数是这个对象的原型，第二个可选参数用以对对象的属性进行描述。

```js
var o1 = Object.create({x:1, y:2});
var o2 = Object.create(null);//创建的对象没有原型，也就是没有任何基础方法，比如 toString() 之类的
```

## 属性的查询和设置

### 属性访问错误

设置属性会失败的场景：

- o 中的属性 p 是只读的：不能给只读属性重新赋值（`defineProperty()` 方法中有一个例外，可以对可配置的只读属性重新赋值）。
- o 中的属性 p 是继承属性，且是只读的：不能通过同名自有属性覆盖只读的继承属性。
- o 中不存在自有属性 p：o 没有使用 setter 方法继承属性 p，并且 o 的可扩展性是 false。如果 o 中不存在 p，且没有 setter 方法可供调用，则 p 一定会添加至 o 中。但如果 o 不是可扩展的，那么在 o 中不能定义新属性。

### 检测属性

- `"x" in o`：包括自有属性和继承属性。
- `o.hasOwnProperty("x")`：包括自有属性。
- `o.propertyIsEnumerable("x")`：包括自有属性，且属性是可枚举的。
- `o.x !== undefined`：同 `in`，但不能区分赋值为 undefined 的属性。
- `o.x != null`：不区分 null 和 undefined。

## 属性的特性

属性的 4 个特性：值 value、可写性 writable、可枚举性 enumerable、可配置性 configurable。
存取器属性没有值和可写性，取而代之的是读取 get、写入 set。

ES3 中无法配置这些特性，ES5 中定义了一个“属性描述符”的对象。

```js
//{value: 1, writable:true, enumerable:true, configurable:true}
Object.getOwnPropertyDescriptor({x:1}, "x");

//{get:[Function: get x], set:undefined, enumerable:true, configurable:true}
Object.getOwnPropertyDescriptor({get x(){}}, "x");

//不存在的属性和继承属性
//undefined
Object.getOwnPropertyDescriptor({}, "x");
Object.getOwnPropertyDescriptor({}, "toString");

//创建或修改属性的特性
var o = {};
var pd = {value:1, writable:true, enumerble:false, configurable:true};
Object.defineProperty(o, "x", pd);//创建属性并定义特性
Object.defineProperty(o, "x", {writable:false});//修改特性
//多个属性的特性，返回修改后的对象
var p = Object.defineProperties({}, {
  x: {value:1, /*...*/},
  y: {value:2, /*...*/}
});
```

任何对 `Object.defineProperty()` 或 `Object.defineProperties()` 违反规则的使用都会抛出类型错误异常：

- 若对象是不可扩展的，则可以编辑已有的自有属性，但不能给它添加新属性。
- 若属性是不可配置的，则不能修改它的可配置性和可枚举性。
- 若存取器属性是不可配置的，则不能修改其 getter 和 setter 方法，也不能将它转换为数据属性。
- 若数据属性是不可配置的，则不能将它转换为存取器属性。
- 若数据属性是不可配置的，则不能将它的可写性从 false 修改为 true，但可以从 true 改为 false。
- 若数据属性是不可配置且不可写的，则不能修改它的值。然而可配置但不可写属性的值是可以修改的（实际上是先将它标记为可写，然后修改值，最后转换为不可写）。

## 对象的三个属性

原型、类、可扩展性

### 原型

- 通过对象直接量创建：Object.prototype。
- 通过 new 构造函数创建：构造函数的 prototype。
- 通过 `Object.create()` 创建：使用第一个参数，可以是 null。

在 ES5 中，通过 `Object.getPrototypeOf()` 查询对象的原型。在 ES3 中没有对应的函数，可以通过 `o.constructor.prototype` 来查询原型，但这种方式并不可靠。

通过对象直接量创建的对象和通过 `Object.create()` 创建的对象都含有 constructor 属性，它们的这个属性都指向构造函数 `Object()`。但只有对象直接量创建的对象的原型是 `Object()` 函数的 prototype 属性，而 `Object.create()` 创建的对象往往不是。

要检测 a 对象是否是 b 对象的原型（或在原型链上），应使用 `b.isPrototypeOf(a)`。

### 可扩展性

可扩展性表示是否可以给对象添加新属性。所有内置对象和自定义对象都是显式可扩展的。

- `Object.isExtensible()`：判断对象是否可扩展。
- `Object.preventExtensions()`：转为不可扩展，然后就无法转回可扩展了。只影响对象本身，依然可以给原型添加属性，此对象仍会继承添加的属性。
- `Object.seal()`：将对象设置为不可扩展，且所有自有属性都不可配置。已封闭的对象不能解封，可用 `Object.isSealed()` 来检测。
- `Object.freeze()`：在 seal 的基础上，还将所有自有数据属性置为只读（若存取器属性有 setter 方法则仍能赋值）。可用 `Object.isFrozen()` 来检测。

## 序列化对象

`JSON.stringify()` 和 `JSON.parse()`

# 数组

## 数组遍历

判断数组中的元素

```js
if (!a[i]); // null、undefined 和不存在的元素
if (a[i] === undefined); // undefined 和不存在的元素
if (!(i in a)); // 不存在的元素
```

由于 for/in 循环会枚举继承的属性，所以对于数组要么不用 for/in 循环，要么手动判断继承的属性。

for/in 循环的遍历顺序不能确定，可以改用传统的 for 循环，或用 ES5 中新增的 forEach() 方法。

```js
var arr = [1,2,3];
arr.forEach(function(x) {
  //operations on the element
});
```

## 数组方法

### join

拼接数组元素，默认使用逗号，也可以自定义连接符。是 string.split() 的逆向操作。

### reverse

反转排列。在原数组中反转，而不是新建一个数组。

### sort

默认按字母表顺序排序，必要时会把元素临时转为字符串进行比较。undefined 元素会被排到末尾。

可传入比较函数，若第一个参数应排在前面，则函数应返回小于 0 的数值。

### concat

返回一个**新数组**，其中包含原有的元素和所有入参。若参数是数组，则添加的是其中的每个元素，而不是将该数组整个作为一个元素。

### slice

返回一个**新数组**，该数组是原数组的一部分。

- 两个入参：两个索引，子数组包含了第一个索引的元素到第二个索引前面一个元素。
- 一个入参：入参表示开始位置，结束位置固定为数组末尾。
- 入参为负数：表示相对于 0 索引的位置。

### splice

对**原数组**进行删除和插入操作。被操作的元素后面的元素会自动调整位置。返回结果是一个数组，包含了被删除的元素，可能为空数组。

- 第一个参数：删除和插入的起始位置。
- 第二个参数：删除的元素个数。
- 第三个开始的所有参数：要插入的元素，若参数为数组，则这个数组本身作为一个元素（这一点与 `concat()` 不同）。

### push 和 pop

在**原数组**的末尾插入和弹出元素。若插入的是数组，则数组本身作为一个元素。

`push()` 返回插入后的数组长度。`pop()` 返回弹出的元素。

### unshift 和 shift

在**原数组**的开始插入和弹出元素。若插入的是数组，则数组本身作为一个元素。

`unshift()` 返回插入后的数组长度。`shift()` 返回弹出的元素。

同时插入多个元素时会一次性插入，而不是一个个插入，也就是说插入后的元素顺序跟参数里的顺序一致。

## ES5 中的数组方法

### forEach

`forEach()` 方法的第一个参数是一个函数。该函数有 3 个参数，依次为：数组元素、元素的索引、数组本身。

`forEach()` 方法只能通过在函数中抛出 `foreach.break` 异常来提前中断。

### map

传入一个映射函数，对每个元素做映射，然后返回一个包含新元素的新数组。

若原数组是稀疏数组，则新数组也是稀疏数组，缺失的元素一样。

### filter

传入一个判断函数，该函数返回 true/false，通过判断的元素会被放入新数组中。

稀疏数组中缺失的元素会被跳过，即返回的数组总是稠密的。

### every 和 some

传入一个判断函数，该函数返回 true/false，对所有元素应用该函数。

- every：仅当所有元素都通过判断时才返回 true。
- some：当至少有一个元素通过判断时就返回 true。

一旦确认返回值就会直接返回。根据数学上的惯例，对于空数组，every 返回 true，some 返回 false。

### reduce 和 reduceRight

传入的第一个参数是一个组合函数，对数组内的元素进行组合，最终返回一个组合后的值。第二个参数是初始值，缺省为第一个元素。

reduce 是按照索引从低到高，reduceRight 则相反。

### indexOf 和 lasIndexOf

在数组中查找指定元素，返回找到的第一个元素的索引，若未找到则返回 -1。

第二个参数是可选的，它指定开始搜索的位置。

## 数组类型

ES5 中可以使用 `Array.isArray()` 来判断对象是否为数组。ES3 要自行判断：

```js
var isArray = Function.isArray || function(o) {
  return typeof o === "object" &&
    Object.prototype.toString.call(o) === "[object Aray]";
};
```

# 函数

## 函数定义

**函数声明**

```
function funcName(params) {/*code*/}
```

**函数表达式**

```
var x = function(a, b) {return a * b};
```

**构造函数**

```
var myFunction = new Function("a", "b", "return a * b");
```

但一般不使用构造函数。

**函数提升（Hoisting）**

“提升”是 JavaScript 默认将当前作用域提升到前面去的行为。
“提升”应用在变量的声明与函数的声明，但不应用于变量和函数的赋值。
因此，函数可以在声明之前调用。
“提升”是在 js 引擎预编译的时候进行的，即运行前。

**函数是对象**

在 JavaScript 中使用 typeof 操作符判断函数类型将返回 "function" 。
但是 JavaScript 函数描述为一个对象更加准确。
JavaScript 函数有 属性 和 方法。
arguments.length 属性返回函数调用过程接收到的参数个数：

```
function myFunction(a, b) {
    return arguments.length;
}
```

`toString()` 方法将函数作为一个字符串返回:

```
function myFunction(a, b) {
    return a * b;
}
var txt = myFunction.toString();
```

## 函数调用

有 4 种方式来调用 JavaScript 函数：

- 作为函数
- 作为方法
- 作为构造函数
- 通过函数的 `call()` 和 `apply()` 方法间接调用

### 函数调用

根据 ECMAScript 3 和非严格的 ECMAScript 5 对函数调用的规定，调用上下文 this 的值是全局对象。而在严格模式下调用上下文则是 undefined。

```javascript
//判断当前是否为严格模式
var strict = (function() { return !this; }());
```

### 方法调用

方法调用的上下文是包含方法的那个对象。

嵌套函数不会从调用它的函数中继承 this。如果嵌套函数作为方法调用，其 this 的值指向调用它的对象。如果嵌套函数作为函数调用，其 this 的值要么是全局对象（非严格模式）要么是 undefined （严格模式）。

若内部函数想访问外层函数的 this，则要在外层函数中把 this 赋给一个变量（通常为 self），然后内部函数访问这个变量。

### 构造函数调用

如果函数或方法调用之前带有关键字 new，就构成构造函数调用。

没有形参的构造函数调用可以省略括号。

构造函数调用新对象，该对象继承自构造函数的 prototype 属性。

构造函数中的 this 指向它新创建的对象，而不是调用构造函数的对象。比如 `new o.m()` 中，this 指向的不是 o。

### 即调函数 IIFE

> Immediately-Invoked Function Expression

**为什么需要 IIFE**

由于 js 在作用域方面比较薄弱，只有全局作用域和方法作用域，直到 ES6 才有块作用域，所以只能通过将逻辑封装到函数声明中来实现作用域隔离。

很多时候，我们需要执行一段逻辑，而这段逻辑很可能只需要执行一次，那么让这段逻辑的方法名和里面变量名去占用全局的命名空间就很不划算，这时候就需要使用 IIFE。

函数表达式可以 "自调用"。
自调用表达式会自动调用。
如果表达式后面紧跟 `()` ，则会自动调用。
不能自调用声明的函数。
通过添加括号，来说明它是一个函数表达式：

```
(function () {
    var x = "Hello!!";
})();

//实际上只要想办法把函数声明转换成表达式就行
(function() {
    var x = "Hello!";
}());
//再比如
!function() {
    var x = "Hello!";
}();
```

虽然各种把函数声明转换成表达式的方式都能生效，但是性能会有差别。

js 引擎为了提升执行速度，在函数声明的时候只会做简单的语法分析，然后在函数执行的时候再做进一步的处理。对于自调函数显然应该直接做完整的处理。然而由于 js 引擎是通过符号特征来识别自调函数的（比如 `(function`），而有些引擎漏掉了一些不常见的特征，从而导致使用这些语法把函数声明转换成表达式的时候引擎不会把它当成自调函数来做优化，因而影响性能。

## 函数参数

**函数显式参数(Parameters)与隐式参数(Arguments)**

> 显示参数在函数定义时给出，相当于形参。
> 隐式参数在函数调用时传递，相当于实参。

**参数规则**

JavaScript 函数定义时显式参数没有指定数据类型。
JavaScript 函数对隐式参数没有进行类型检测。
JavaScript 函数对隐式参数的个数没有进行检测。

### 实参对象

实参对象 `arguments` 不是一个数组，而是一个对象，只是它的属性碰巧是以数字为索引。

在非严格模式下，`arguments` 中的属性和对应的实参指向的是同一个对象，即修改时两处会同时改变。（ES5 中移除了此特性）

**callee 和 caller 属性**

实参对象还定义了 `callee` 和 `caller` 属性。

在 ECMAScript 5 严格模式下，对这两个属性读写会产生类型错误。在非严格模式下，ES 规范规定 `callee` 属性指代当前正在执行的函数，而 `caller` 是非标准的，大多数浏览器实现这个属性指代调用当前执行函数的那个函数。


## 函数属性、方法和构造函数

### call() 方法和 apply() 方法

第一个参数作为方法中 `this` 的值，后面的参数作为方法的实参。

在 ES5 的严格模式中，第一个参数即使是原始类型或者 null 或者 undefined 也会作为 `this` 的值。在 ES3 和非严格模式中，原始类型会被包装对象替代，null 和 undefined 会被全局对象替代。

call() 从第二个参数开始依次作为函数的实参。apply() 将函数的所有实参放入一个数组中作为第二个参数。

### bind() 方法

bind() 方法的第 1 个参数是一个对象，它会被作为被绑定方法的 this。从第 2 个参数开始是可选的，它们依次绑定到被绑定方法的入参上。

```js
var sum = function(x, y) { return x + y + this.z; };
var newSum = sum.bind({z:3}, 1);

var result = newSum(2);// x=1,y=2,z=3
```

通常被用来实现科里化。

# 类和模块

## 类和构造函数

构造函数的 prototype 属性被用作新对象的原型。

```js
//构造函数首字母大写，这是编程约定而不是语法限制。
function Range(from, to) {
  this.from = from;
  this.to = to;
}

Range.prototype = {
  includes: function(x) { /*...*/ },
  foreach: function(f) { /*...*/ },
  toString: function() { /*...*/ }
};

var rangeObj = new Range(1, 2);
```

### 构造函数和类的标识

两个对象是否为同一个类的实例，取决于它们的 prototype 是否指向同一个对象，而不是取决于它们是否从同一个构造函数创建的。

### constructor 属性

因为任何函数都可以作为构造函数，而调用构造函数是需要用到 prototype 属性的，所以每个函数（除 ES5 中的 `Function.bind()`）都自动拥有一个 prototype 属性。该属性包含唯一一个不可枚举属性 constructor，它的值是一个函数对象。

像上述那样显式定义 Range.prototype 的话就不存在 constructor 属性了，需要显式定义。

另一种方式是在预设的 prototype 上添加方法：

```js
Range.prototype.includes = function(x) { /*...*/ };
Range.prototype.foreach = function(f) { /*...*/ };
```

## Javascript 中的 Java 式类继承

# 脚注

[^1]:Unicode 对其所有字符做了分类，这种分类用“通用类别值”表示，这里的“Mn”、“Mc”和“Pc”分别表示基字符的修改中出现的非间距字符、基字符的修改中影响了基字符标识位的宽度的间距字符、连接两个字符的连接符或标点符号。
