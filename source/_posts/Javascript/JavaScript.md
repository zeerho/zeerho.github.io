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

## 属性 getter 和 setter

在 ES5 中，属性值可以由 getter 和 setter 方法来代替，这种属性称为“存取器属性”。

```js
var o = {
  $n = 0; // $ 暗示是私有属性
  get next() {
    return this.$n++;
  }
  set next(n) {
    if (n >= this.$n) {
      this.$n = n;
    } else {
      throw "blabla";
    }
  }
}
```

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

- 构造函数对象：构造函数为类定义了名字。任何添加到构造函数对象中的属性都是类属性或类方法。
- 原型对象：原型对象的属性被类的所有实例继承。
- 实例对象：添加到实例对象中的属性是该对象独享的。

## 类和类型

### instanceof

instanceof 右边的参数是构造函数，但计算过程实际上是检测了对象的原型继承关系，而不是创建对象的构造函数。

也可以不用构造函数，而直接用对象来判断继承关系：`examplePrototype.isPrototypeOf(obj)`。

这两种方法的缺点是：

- 只能检测对象是否属于某个类，而不能在类未知的情况下通过对象获取类名。
- 在 Web 客户端中，每个窗口和框架子页面都有独立的上下文，概念上相同的类/对象在不同上下文中都是独立的。

### 构造函数的名称

为解决不同上下文中的类型检测问题，可以通过构造函数名称来区分。

```js
function type(o) {
  var t, c, n; // type, class, name
  if (o === null) return "null";
  if (o !== o) return "NaN"
  if ((t = typeof o) !== "object") return t; // 原始值的类型和函数
  if ((c = classof(o) !== "Object") return c; // 可以识别出大多数的内置对象
  if (o.constructor && typeof o.constructor === "function" && (n = o.constructor.getName())) return n; // 若构造函数名字存在的话
  return "Object";
}

// 返回对象的类
function classof(o) {
  return Object.prototype.toString.call(o).slice(8, -1);
};

// 返回函数的名字（可能是空字符串），不是函数的话返回 null
Function.prototype.getName = function() {
  if ("name" in this) return this.name;
  return this.name = this.toString().match(/function\s*([^(]*)\(/)[1];
}
```

这种方法的问题在于，不是所有对象都有 constructor 属性，也不是所有函数都有名字，比如：

```js
// 这个构造函数没名字
var Complex = function(x, y) { /*...*/ };
var Range = function Range(f, t) { /*...*/ };
```

### 鸭式辩型

> 不要关注“对象的类是什么”，而是关注“对象能做什么”。

## Javascript 中的面向对象技术

### 标准转换方法

- toString：在希望使用字符串的地方（比如将对象用作属性名或用“+”拼接字符串），JavaScript 会自动调用此方法。
- toLocaleString：默认情况下 toLocalString 方法只是简单的调用了 toString 方法。
- valueOf：用来将对象转换为原始值。比如当数字运算符（除了“+”）和关系运算符作用于数字文本表示的对象时，会自动调用 valueOf。
- toJSON：由 JSON.stringify() 自动调用，用来进行对象序列化。

### 私有状态

通过将变量（或参数）闭包在一个构造函数内来模拟私有实例字段。但是这样会降低运行速度、占用更多内存。

```js
funtion Range(from, to) {
  this.from = function() { return from; };
  this.to = function() { return to;};
}
```

# 正则表达式的模式匹配

```js
var pattern = /abc/;
// 或者
var pattern = new RegExp("abc");
```

一些 Perl 正则语法不被 ECMAScript 支持：

- s（单行模式）和 x（扩展语法）标记。
- \a、\e、\l、\u、\L、\U、\E、\Q、\A、\Z、\z、\G 转义字符。
- `(?<=` 正向后行断言、`(?<!` 负向后行断言。
- `(?#` 注释和扩展 `(?` 的语法。

## 正则表达式的定义

### 修饰符

- i：不区分大小写。
- g：全局匹配。
- m：多行匹配。`^` 匹配字符串开头和换行后的开头，`$` 同理。

## 用于模式匹配的 String 方法

`"abc".search(/ab/);`

传入正则表达式（若不是则自动通过 RegExp 构造函数转换），返回 -1（匹配不到）或匹配结果的首字符的索引。不支持全局搜索（即使使用修饰符 g）。

`"abc".replace(/ab/, "cd");`

传入两个参数。第一个参数是正则表达式（若不是则自动通过 RegExp 构造函数转换），第二个参数是要替换的字符串或是用来计算字符串的函数。第二个参数中可以通过 `$` 加数字来引用分组。

`"abc".match(/ab/);`

传入正则表达式（若不是则自动通过 RegExp 构造函数转换），返回由匹配结果组成的数组。需要在表达式中使用修饰符 g 来获取全部匹配结果。若是全局匹配，则结果数组包含了所有匹配。若不是全局匹配，则 a[0] 是完整的匹配，a[1] 开始的元素依次是每个分组的匹配。

`"abc".split(/a/);`

入参可以是字符串或正则表达式。返回分割之后的数组。

## RegExp 对象

构造函数接受一个必选参数，一个可选参数。

必选参数是字符串形式的正则表达式。可选参数是字符串形式的修饰符。

### RegExp 的属性

- source：只读的字符串，包含正则表达式的文本。
- global：只读的布尔值，说明是否带有修饰符 g。
- ignoreCase：只读的布尔值，说明是否带有修饰符 i。
- multiline：只读的布尔值，说明是否带有修饰符 m。
- lastIndex：可读/写的整数，如果带有修饰符 g，则这个属性存储在整个字符串中下次检索的开始位置。

### RegExp 的方法

**exec()**

传入一个字符串。如果没有匹配，则返回 null；如果有一个匹配，则返回一个数组，a[0] 是匹配结果，a[1] 开始依次是分组匹配，index 属性包含了发生匹配的字符位置，input 属性包含了正在检索的字符串。跟 `String.match()` 不同，不管有没有修饰符 g，都会返回一样的数组。

若带有修饰符 g，则可以反复调用此方法来尝试获得多个匹配。此方法会通过设置正则表达式的 lastIndex 属性来实现此功能。

**test()**

和 `exec()` 等价，当 `exec()` 返回不是 null 时，此方法返回 true。如带有修饰符 g，则也可以反复调用此方法来尝试进行多次匹配。

**与 String 的方法的区别**

String 的 `search()`、`replace()`、`match()` 方法不会用到 lastIndex 属性，它们只是简单地将该属性置为 0。如果让一个全局正则表达式对多个字符串执行 `exec()` 和 `test()` 方法，要么找出所有匹配以便将 lastIndex 属性置为 0，要么手动进行设置，否则会影响后续匹配的起始位置。


# JavaScript 的子集和扩展

## JavaScript 的子集

### 子集的安全性

为了让 js 代码静态地通过安全检查，必须移除一些特性：

- `eval()` 和 `Function()` 构造函数在任何安全子集里都是禁止的，因为它们可以执行任意代码，无法对这些代码做静态分析。
- 禁止使用 `this`，因为在非严格模式中，函数可以通过 `this` 访问全局对象。而沙箱系统的一个重要目标就是禁止对全局对象的访问。
- 禁止使用 `with`，因为会增加静态代码检查的难度。
- 禁止使用某些全局变量。`window` 和 `document` 对象可以操作浏览器和页面，因而存在隐患。一种方法是完全禁用这些全局对象，提供自定义的一组 API 来进行有限制的操作。另一种方法是在沙箱代码运行的“容器”内定义一个只对外提供安全的标准 DOM API 的 facade 或 proxy。
- 禁止使用某些属性和方法，以免沙箱中的代码拥有过多权限。包括 `arguments` 的 `caller` 和 `callee` 属性（甚至是 `arguments` 本身）、函数的 `call()` 和 `apply()`，以及 `constructor` 和 `prototype` 属性。
- 相对于 `.` 运算符访问属性而言，`[]` 中的字符串表达式无法做静态分析。因此，安全子集通常禁止使用方括号，除非里面是数字或字符串直接量。安全子集将 `[]` 替换为全局函数，这些全局函数在存取属性之前会执行运行时检查以确保不会读写那些禁止访问的属性。

一些比较重要的安全子集实现：

- ADsafe：只包含静态检查，使用 JSLint 作为检验器。禁止访问大部分全局变量，并定义了一个 `ADSAFE` 变量，提供了一组安全的 API，包括一些特殊的 DOM 方法。
- dojox.secure：基于静态检查，静态检查受限于子集范围内。使用标准 DOM API。同时，它包含一个用 JavaScript 实现的检查器。
- Caja：它定义了两个语言子集。Cajita （小沙盒）是一个与 ADsafe 和 dojox.secure 相似的严格子集。Valija (手提箱或行李箱）范围更广，接近 ECMAScript 5 的严格模式（不含 eval()）。Caja 本身也是一个编译器的名字，可以将一段网页内容转换为一个安全的模块
- FBJS：它是 JavaScript 的变种，依赖代码转换来保证代码的安全性，转换器同时提供运行时检查，以避免通过 `this` 访问全局对象，并对所有顶层标识符重命名，给它们增加了一个标识模块的前缀。因为这种重命名，任何对全局变量和其他模块的变量的操作都无法进行。FBJS 模拟实现了 DOM API 的一个安全子集。
- Microsoft Web Sandbox：定义了 JavaScript 的一个更宽泛的子集，包含 HTML 和 CSS，它的代码重写规则非常激进，有效地重新实现了一个安全的 JavaScript 虚拟机，针对不安全的 js 顶层代码进行处理。

## 常量和局部变量

在 js 1.5 及以后版本中可以用 `const` 定义常量。对常量重新赋值会失败但不会报错，重新声明会报错。常量会被提升至函数顶部。`const` 是 js 保留字，所以无需加入版本号。

`const` 类似 Java 中的 `final`，只能保证引用不被修改，不保证引用的对象内部不被修改。

js 1.7 针对没有块级作用域的缺陷增加了 `let` 关键字，由于不是保留字，所以需要手动加入版本号。

这里的版本号指的是 Mozilla 的语言版本。在 Spidermonkey 和 Rhino 解析器和 Firefox Web 浏览器中实现了这些语言版本。

在 Spidermonkey 或 Rhino 中，可以通过命令行选项指定版本，或通过一个内置函数 `version()` 指定。指定的版本号是实际版本号乘以 100。在 Firefox 中，可以在 script 标签中指定版本：`<script type="application/javascript; version=1.8">`。

`let` 4 种使用方式：

- 作为变量声明，和 `var` 一样。
- 在 for 循环种，作为 `var` 的替代方案。
- 在语句块中定义一个新变量并显式指定它的作用域。
- 定义一个在表达式内部作用域中的变量，只在表达式内可用。

其中第 4 种用法：

```js
//let 语句块
let x=1, y=2;
let (x=x+1, y=x+1) {
  console.log(x+y); //5
};
console.log(x+y);   //3

//------------------------

//let 表达式
let x=1, y=2;
console.log(let(x=x+1, y=x+2) x+y);//5
```

`let` 语句中的变量初始化表达式不是语句块的一部分，并且是在作用域外部解析的。

在 ES6 之前，全局变量会自动挂构成顶层对象（比如浏览器中的 window）的属性，反之亦然。ES6 规定，`let`、`const`、`class` 声明的变量不属于顶层对象的属性。

## 解构赋值

Spidermonkey 1.7 实现了一种混合式赋值，即“解构赋值”。

```js
let [x, y] = [1, 2]; // let x=1, y=2
let [,x,,y] = [1, 2, 3, 4]; // let x=2, y=4

//解构赋值运算的返回值是右侧的整个数据结构，而不是提取出来的某个值
let a, b, all;
all = [a, b] = [1, 2, 3, 4]; // a=1, b=2, all=[1, 2, 3, 4]

//对于数组嵌套的情况，左侧应当是同样格式的嵌套数组
let [a, [b, c]] = [1, [2, 3], 4]; // a=1, b=2, c=3

//对象
let {name:a, age:b} = {name: "tom", age: 18};
console(a + " " + b); // tom 18
```

左侧多余的变量赋值为 undefined，右侧多余的变量被忽略。

## 迭代

### for/each 循环

for/each 循环是由 E4X 规范（ECMAScript for XML）定义的一种新的循环语句。遍历范围包含继承来的可枚举属性。

```js
let o = {a: 1, b: 2, c: 3};
for (let p in o) {
  console.log(p); // a b c
}
for each (let p in o) {
  console.log(p); // 1 2 3
}

let arr = ['a', 'b', 'c'];
for (let p in arr) {
  console.log(p); // 0, 1, 2
}
for each (let p in arr) {
  console.log(p); // a b c
}
```

### 迭代器

- 迭代器必须包含 `next()` 方法，返回集合中的下一个值。
- 当没有下一个值时，会抛出 `StopIteration`。它是 JavaScript 1.7 中的全局对象的属性。

一般不直接用迭代器，而是用可迭代对象。

- 可迭代对象必须包含 `_iterator_()` 方法，该方法返回一个迭代器。
- JavaScript 1.7 的 for/in 循环会自动调用 `_iterator_()` 方法，并处理 StopIteration 异常。

JavaScript 1.7 中有一个全局函数 `Iterator()`。

- `Iterator()` 会自动调用参数的 `_iterator_()`，并返回该迭代器。
- 若参数没有 `_iterator_()` 方法，则会返回参数的一个自定义迭代器。该迭代器每次返回一个包含两个值的数组，第一个元素是属性名，第二个是对应的值。
  - 该迭代器只对自有属性进行遍历，忽略继承属性。
  - 如果传入 `Iterator()` 的第二个参数是 true，返回的迭代器只对属性名进行遍历，忽略属性值。

```js
for (let [k, v] in Iterator({a: 1, b: 2})) {
  console.log(k + "=" + v); // a=1 b=2
}
```

### 生成器

生成器是 JavaScript 1.7 中的特性。

任何使用关键字 `yield` 的函数（哪怕不可达）都称为生成器函数。生成器函数通过 `yield` 返回值，通过 `return` 来终止函数而不带返回值。对生成器函数的调用不是执行函数体本身，而是返回一个生成器对象。

生成器是一个迭代器对象，表示生成器函数当前的执行状态。它有几个方法：

- `next()`：该方法可恢复生成器函数的执行，直到下一条 `yield` 语句，`yield` 的返回值就是 `next()` 的返回值，而 `return` 会使 `next()` 抛出一个 `StopIteration`。
- `close()`：用来释放生成器对象，相当于在挂起位置执行了 `return`。
- `send()`：`yield` 是一个表达式而不是语句，也就是说它是可以有值的。`send()` 和 `next()` 一样继续执行生成器，不同的是它可以传入一个参数作为 `yield` 表达式的值。
- `throw()`：继续执行生成器，可以传入一个参数，`yield` 表达式会将该参数作为异常抛出。

### 数组推导

语法：`[expression for (variable in object) if (condition)]`

数组推导包含三部分：

1. 一个没有循环体的 for/in 或 for/each 循环。其中，变量之前没有 `var` 或 `let`，实际上是使用了隐式的 `let`，因而不会覆盖外部的变量。
2. 在执行循环获得对象之后，用 if 条件判断，若为 true 则继续。if 判断是可选的。
3. 将变量用 expression 计算，并将结果插入要创建的数组中。

### 生成器表达式

在 JavaScript 1.8 中，将数组推导中的方括号替换成圆括号就成了一个生成器表达式。该表达式的值是一个生成器对象，而不是数组。

## 函数简写

JavaScript 1.8 中，函数有一种简写形式。如果函数只是计算并返回一个表达式的值，那么可以写成：`function(arg) expression`。

## 多 catch 从句

从 JavaScript 1.5 开始支持

```js
try {
  //...
} catch (e if e instanceof Example) {
  //...
} catch (e if e === "blabla") {
  //...
} finally {
  //...
}
```

## E4X:ECMAScript for XML

E4X 是 JavaScript 的一个标准扩展。通常用于服务端，大部分客户端还没支持。

# web 浏览器中的 JavaScript

## JavaScript 程序的执行

### 客户端 Javascript 时间线

以下是理想的事件线，但不是所有浏览器都完全实现。

1. 浏览器创建 `Document` 对象，并开始解析页面：在解析 HTML 元素和它们的文本内容后添加 `Element` 对象和 `Text` 节点到文档中。此阶段 `document.readystate` 属性是“loading”。
2. 当 HTML 解析器遇到没有 `async` 和 `defer` 属性的 `<script>` 元素时，它把元素添加到文档中并执行相应脚本。脚本的执行是同步的，在脚本下载（若需要）和执行时解析器会暂停。因此脚本可以通过 `document.write()` 向输入流写入文本。同步脚本可以看到它自身和之前的文档内容。
3. 当遇到 `async` 属性的 `<script>` 元素时，会下载脚本并继续解析文档。脚本会在下载完成后尽快执行，但解析器不会为它暂停。异步脚本禁止使用 `document.write()`。它可以看到自身和之前的文档内容。
4. 当文档解析完成，`document.readystate` 变为“interactive”。
5. 按出现顺序依次执行所有 `defer` 属性脚本。它能访问完整的文档树，但不能使用 `document.write()`。
6. 浏览器在 `Document` 对象上触发 `DOMContentLoaded` 事件，标志着程序执行从同步阶段转到了异步事件驱动阶段。此时可能还有异步脚本没有执行完成。
7. 至此文档已解析完毕，但可能还有其他内容没载入完成，比如图片。当所有内容载入完成、异步脚本执行完成后，`document.readystate` 属性变为“complete”，浏览器触发 `Window` 对象上的 `load` 事件。
8. 开始调用异步事件。

## 兼容性和互用性

### Internet Explorer 里的条件注释

条件注释不是标准规范，而是 IE5 开始引入的功能，主要用于解决 IE 相关的兼容性问题。

```html
<!--[if IE 6]>
只会在 IE6 显示。
<![endif]-->

<!--[if lte IE 7]>
在 IE5、6、7 中显示。还可以用 lt、gt、gte。
<![endif]-->

<!--[if !IE]><-->
不会在 IE 中显示。
<!--><![endif]-->
```

IE 的 JavaScript 解释器也支持条件注释。

```js
/*@cc_on
  @if (@_jscript)
    alert("in IE");//会在 IE 中执行
  @end
  @*/
```

# Window 对象

## 浏览器定位和导航

`Window` 对象和 `Document` 对象的 `location` 属性引用同一个 `Location` 对象，表示当前显示文档的 URL。

`Document` 对象还有一个 `URL` 属性，是文档首次载入后保存文档 URL 的静态字符串。如果定位到文档中的片段标识符，`Location` 对象会相应更新，而 `document.URL` 则不会。

### 解析 URL

`window.location` 引用的是 `Location` 对象，表示当前显示文档的 URL。

`location.href` 是一个字符串形式的 URL。

`Location` 对象的 `toString()` 返回 `href` 的值。

`Location` 对象还有其他属性：`protocol`、`host`、`hostname`、`port`、`pathname`、`search`，分别表示 URL 的各部分。

`Location` 对象的 `hash` 属性（若存在的话）返回 URL 中的片段标识符部分。`search` 属性返回问号之后的 URL（含问号）。

### 载入新的文档

在 `Location` 对象中：

- `assign()` 使窗口载入指定 URL 的文档。
- `replace()` 类似 `assign()`，只是在载入新文档之前会从浏览历史中删除当前文档。
- 另一种更传统的跳转方法是把新的 URL 赋给 location` 属性。
- `reload()` 重现载入当前文档。

## 浏览历史

`window.history` 属性引用的是 `History` 对象。`History.length` 表示浏览历史列表中的元素数量。出于安全考虑，脚本不能访问已保存的 URL，否则任意脚本都能窥探浏览历史。

`back()` 和 `forward()` 跟浏览器的后退、前进一样。`go()` 接受一个整数参数，在历史列表中向前（正整数）或向后（负整数）跳跃指定页数。

如果窗口包含多个子窗口（比如 `<iframe>`），子窗口的历史会按时间顺序穿插在主窗口的历史中。

## 浏览器和屏幕信息

### Navigator 对象

`window.nevigator` 引用的是包含浏览器厂商和版本信息的 `Navigator` 对象（此名字是为了纪念 Netscapte Nevigator 浏览器）。

- `appName`：Web 浏览器的全称。在 IE 中是“Microsoft Internet Explorer”，在其他浏览器中通常是“Netscape”。
- `appVersion`：通常以数字开始，跟着包含浏览器厂商和版本信息的详细字符串。字符串没有标准格式，不能直接用来判断浏览器的类型。
- `userAgent`：浏览器在 `USER-AGENT` 头部中发送的字符串。
- `platform`：运行浏览器的操作系统的字符串。
- `onLine`：若存在的话表示浏览器当前是否连接到网络。
- `geolocation`：用于确定用户地理位置。
- `javaEnabled()`：非标准方法，判断浏览器是否可以运行 Java 小程序。
- `cookieEnable()`：非标准方法，判断浏览器是否可以保存永久的 cookie。当 cookie 配置为“视具体情况而定”时可能返回不正确的值。

### Screen 对象

- `width`、`height`：以像素为单位的窗口大小。
- `availWidth`、`availHeight`：实际可用的大小。
- `colorDepth`：显示的 BPP（bits-per-pixel）值，典型的值是 16、24、32。

## 对话框

- `alert()`：提示框。
- `confirm()`：确认框，返回布尔值。
- `prompt()`：提示输入框，返回用户输入的字符串。
- `showModalDialog()`：显示一个包含 HTML 格式的“模态对话框”，可以给它传入参数，以及从对话框里返回值。

## 错误处理

`Window.onerror` 的值是一个事件处理程序。由于历史原因，它接受三个字符串参数而不是一个事件对象。

1. 描述错误的一条消息。
2. 引发错误的 js 代码所在文档的 URL 的字符串。
3. 文档中发生错误的行数。
4. 返回 false，表示它通知浏览器事件处理程序已经处理了错误，即浏览器不应该显示错误消息。但由于历史原因，Firefox 里的错误处理程序必须返回 true 来表示它已经处理了错误。

## 多窗口和窗体

### 打开和关闭窗口

`Window.open()` 有 4 个可选参数：

1. 新窗口中文档的 URL，若省略或为空字符串，则会使用 about:blank。
2. 新窗口的名字。若指定的是已存在的窗口名字，则会直接使用该窗口。若省略会使用 `_blank`。
  - 只有设置了“允许导航”（allowed to navigator）（HTML5 术语）的页面才行。当且仅当窗口包含的文档是同源的，或是此脚本打开了那个窗口（包括递归打开），脚本才能只通过名字来指定存在的窗口。
  - 若两个窗口是内嵌关系，那它们的脚本可以互相导航。此时可以用保留的名字 `_top` 和 `_parent` 来获取彼此的上下文。
3. 以逗号分隔的列表，包含大小和各种属性。若省略则会用一个默认的大小，而且带有一整组标准的 UI 组件，即菜单栏、状态栏、工具栏等。这个参数是非标准的。
4. 一个布尔值，只在第二个参数命名的是一个存在的窗口时才有用，声明了由第一个参数指定的 URL 是应该替换掉窗口浏览历史的当前条目（true），还是应该在历史中创建一个新的条目（false），后者是默认值。

返回值是代表命名或新创建的窗口的 `Window` 对象。该对象的 `opener` 属性引用的是调用 `open` 方法的那个窗口。

### 窗体之间的关系

顶级窗口或标签的 `parent` 和 `top` 属性引用的是自身。

`<iframe>` 元素由 `contentWindow` 属性，引用的是该窗体对象。反过来，窗体的 `window.frameElement` 属性引用的是相应的 `<iframe>` 元素对象。顶级窗口对象的 `frameElement` 属性为 null。

通常用另一种方式来获取其他窗体对象。

每个 `Window` 对象都有一个 `frames` 属性，包含了该窗口所包含的窗口或窗体的 `Window` 对象（而不是 `<iframe>` 对象）。

```js
//第二个子窗体的第三个子窗体。
var w1 = frames[1].frames[2];
//兄弟窗体
var w2 = parent.frames[1];
//通过 <iframe> 元素的 name 或 id 来索引
var w3 = frames["f1"];
```

实际上 `frames` 属性引用的是 `window` 本身，也就是说 `window` 本身就是一个由子窗体对象组成的数组。不过通过 `frames` 来引用会显得清楚一点。

# 脚本化文档

## 文档结构和遍历

### 作为节点树的文档

`Document`、`Element`、`Text` 对象都是 `Node` 对象。`Node` 定义了以下属性：

- `parentNode`：若无父节点则为 null。
- `childNodes`：只读的类数组对象（`NodeList`）。
- `firstChild`、`lastChild`
- `nextSibling`、`previousSibling`
- `NodeType`：节点类型。9：Document，1：Element，3：Text，8：Comment，11：DocumentFragment。
- `NodeValue`：Text 节点或 Comment 节点的文本内容。
- `nodeName`：元素的标签名，大写。

# Javascript核心参考

## String

**属性**

- `length`: 字符数。

**方法**

- `charAt` 返回指定索引处的字符。
- `charCodeAt` 返回指定索引处的字符的编码。
- `concat` 将若干值拼接成一个字符串。
- `indexOf` 返回子字符串第一次出现的位置。
- `lastIndexOf` 返回子字符串最后一次出现的位置。
- `localeCompare` 使用本地定义的顺序比较字符串。
- `match` 使用正则进行匹配。
- `replace` 使用正则进行替换。
- `search` 返回匹配正则的子字符串。
- `slice(start, end)` 返回一个子字符串。
  - `start` 起始处索引（包含），可为负数。
  - `end` 结束处索引（不含），可为负数。若未指定则默认为结尾。
- `split` 用指定字符串或正则表达式作为分隔符，返回分割后的字符串数组。
- `substr(start, length)` 返回一个子字符串。指定起始索引和子字符串长度。
- `substring(start, end)` 返回一个子字符串。索引不能为负数。
- `toLowerCase` 转小写。
- `toUpperCase` 转大写。
- `trim` 删除前后空白字符。

**静态方法**

- `fromCharCode` 创建指定编码对应的字符串。

### replace

> string.replace(regex, replacement)

- `regex` 正则表达式对象。若实际传入的是字符串而不是正则对象，则按照字符串字面量匹配，而不会自动转换成正则对象。
- `replacement` 替换用的字符串或函数。
  - 若为函数（ES3 开始），则函数的入参是：
    - 匹配到字符串，它的返回值会作为替换用的字符串。
  - 若为字符串，则其中的 `$` 有特殊含义。
    - `$1`, `$2`, ..., `$99`: 对应 `regex` 中的分组内匹配到的文本。
    - `$&`: `regex` 匹配到的子串。
    - `$`` `: 子串的左边文本。
    - `$'`: 子串的右边文本。
    - `$$`: 美元符号本身。



# 脚注

[^1]:Unicode 对其所有字符做了分类，这种分类用“通用类别值”表示，这里的“Mn”、“Mc”和“Pc”分别表示基字符的修改中出现的非间距字符、基字符的修改中影响了基字符标识位的宽度的间距字符、连接两个字符的连接符或标点符号。

