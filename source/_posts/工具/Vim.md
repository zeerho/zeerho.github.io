---
title: Vim
date: 2017-01-01 09:00:00
tags: [工具, vim]
---

# 基本命令

## 撤销

- `u` 撤销从上次进入插入模式到退出插入模式之间的操作。在插入模式中使用光标键会视为产生一个新的撤销块。
- `U` 撤销对整行的修改
- `<c-r>` 重做被撤销的操作

## 更改

- `[n]<C-a>` 将光标所在数字（或本行内光标后最近一个数字）加 n（缺省为 1）
- `[n]<C-x>` 将光标所在数字（或本行内光标后最近一个数字）减 n（缺省为 1）

Vim 缺省会将以 0 开头的数字解释为八进制。在 vimrc 中加入 `set nrformats=` 来把所有数字解释为十进制。

## 插入模式下的删除操作符

|按键操作 |用途                      |
|:--------|:-------------------------|
|`<C-h>`  |删除前一个字符（同退格键）|
|`<C-w>`  |删除前一个单词            |
|`<C-u>`  |删至行首                  |

这些命令在 Vim 的命令行模式以及在 bash shell 中同样有效。

## 文件状态

- `<c-g>` 显示光标所在行位置及文件状态信息

## 搜索

- `/{string}` 正向查找指定字符串
- `?{string}` 反向查找指定字符串
- `n` 以相同方向再次查找该字符串
- `N` 以相反方向再次查找该字符串
- `<c-o>` 返回之前位置
- `<c-i>` 跳至较新位置
- `*` 查找光标下单词

### 正则表达

`/expr` magic 搜索模式。
`/\vexpr` very magic 搜索模式。除下划线、字母和数字以外的所有字符都当作具有特殊含义的字符。“#”依然会按其原义匹配，原因是“任何还未具有特殊含义的字符都被保留以备将来扩展时使用”。
`/\Vexpr` very nomagic 搜索模式。只有“\”具有特殊含义。
*具体的字符含义见 `:h pattern-overview`*

`/\(expr1\)\@>expr2` 固化分组。详见另一篇《正则表达式》。
`/expr1\(expr2\)\@=` 正向前查找。匹配所有满足后置表达式为 expr2 的 expr1。
`/expr1\(expr2\)\@!` 负向前查找。匹配所有满足后置表达式**不**为 expr2 的 expr1。
`/expr1\@<=expr2` 正向后查找。匹配所有满足前置表达式为 expr1 的 expr2。
`/expr1\@<!expr2` 负向后查找。匹配所有满足前置表达式**不**为 expr1 的 expr2。
*若用 very magic 模式则不需要转义*

`*` 贪婪匹配
`{-}` `{-m,n}` 非贪婪匹配

### 配对括号的查找

`%` 若光标置于括号处（{、[、(、)、]、}），则会跳至与该括号相配对的另一括号处。若光标置于非括号处，则会跳至与“当前位置之后第一个括号”相配对的另一括号处（仅在本行查找）。

## 在 VIM 内执行外部命令

`:![command]` 所有的外部命令都可以用这种方式执行，包括带命令行参数的命令。

## 具有选择性的保存命令

1. 按 `v` 进入可视模式
2. 移动光标来选取文本
3. 按 `:` 会看到窗口底部出现 `:'<,'>`
4. 输入 `w [FILENAME]` 并回车，就把所选文本保存到相应的文件中了。

## 提取和合并文件

`:r [FILENAME]` 插入指定文件的内容，从光标所在位置的后一行开始
`:r ![command]` 插入指定外部命令的输出内容

## 设置类选项

- `:set ic` 查找替换忽略大小写 *Ignore Case*
- `:set noic` 禁用忽略大小写
- `:set hls` 搜索高亮 *hlsearch*
- `:set nohls` `:nohls` `:nohlsearch` 禁用搜索高亮
- `:set is` 增量搜索 *incsearch*
- `:set nois` 禁用增量搜索
- `/[string]\c` 仅在本次查找忽略大小写

## 获取帮助信息

`<help>` 若键盘上有的话
`<F1>` 若键盘上有的话
`:help [parameter]` 获取帮助信息，参数为可选项

## 常用变量

- `$myvimrc`: vimrc 文件全路径。（关于 vimrc 详见 `:h vimrc-intro`）
- `$vim`: vim 程序根目录。
- `$vimruntime`: 其下包含各种 vim 的支持文件。以 vim8.1 为例，默认是 `$vim/vim81/`。


## 其他

`gv` 重选上次的选区
`o` 切换选区的活动端

# mswin.vim

`{mode} {if_recursive} {map}`

- `mode` 表示该映射生效的模式（*`:h vim-modes`*）。取值如下：
    - 空: normal, visual, select, operator-pending
    - n: normal only
    - v: visual and select
    - o: operator-pending(*`:h Operator-pending-mode`*)
    - x: visual only(*`:h visual-mode`*)
    - s: select only(*`:h select-mode`*)
    - i: insert
    - c: command-line
    - l: insert, command-line, regexp-search (and others. Collectively called "Lang-Arg" pseudo-mode)
- `if_recursive` 表示该命令是否会递归生效
    - ` ` 空缺，表示递归生效
    - `nore` 表示非递归生效
- `map` 固定值，表示映射

# 可重复的操作及如何回退

|目的                    |操作               |重复 |回退 |
|:-----------------------|:------------------|:----|:----|
|做出一个修改            |{edit}             |`.`  |`u`  |
|执行替换                |:s/{tar}/{sub}     |`&`  |`u`  |
|执行一系列修改          |`qx{changes}q`     |`@x` |`u`  |
|重复任意 Ex 命令        |`:{Ex}`            |`@:` |`u`  |
|在行内查找下一指定字符  |`f{char}`/`t{char}`|`;`  |`,`  |
|在行内查找上一指定字符  |`F{char}`/`T{char}`|`;`  |`,`  |
|在文档中查找下一处匹配项|/pattern`{CR}`     |`n`  |`N`  |
|在文档中查找上一处匹配项|?pattern`{CR}`     |`n`  |`N`  |


# 操作符 operator

|命令 |用途            |
|:----|:---------------|
|`c`  |修改            |
|`s`  |光标下单字符修改|
|`d`  |删除            |
|`y`  |复制到寄存器    |
|`g~` |反转大小写      |
|`gu` |转换为小写      |
|`gU` |转为大写        |
|`>`  |增加缩进        |
|`<`  |减小缩进        |
|`=`  |自动缩进        |
|`!`  |使用外部程序过滤 {motion} 所跨越的行|

*自定义操作符`:h map-operator`*

# Ex 命令

- `:[{range}]d[elete] {reg}` 删除指定范围内的到寄存器 {reg} 中
- `:[{range}]y[ank] {reg}` 复制指定范围内行[到寄存器 {reg} 中
- `:[{line}]pu[t] {reg}` 在指定行后粘贴寄存器 {reg} 中的内容
- `:[{range}]co[py] {to}` `:t` 把指定范围内的行复制到 {to} 所指定的行之下
- `:[{range}]m[ove] {to}` 把指定范围内的行移动到 {to} 所指定的行之下
- `:[{range}]j[oin][!]` 连接指定范围内的行。`!` 表示不插入或删除任何空格。
- `:[{range}]norm[al][!] {commands}` 对指定范围内的每一行执行普通模式命令
- `:[{range}]s[ubstitute]/{pattern}/{string}/[flags]` 把指定范围内出现 {pattern} 的地方替换为 {string}
- `:[{range}]g[lobal]/{pattern}/[cmd]` 对指定范围内匹配 {pattern} 的所有行，在其上执行 Ex 命令 {cmd}

## 补全功能

1. 确保 Vim 不是在以兼容模式运行 `:set nocp`
2. `<C-d>` 列出候选列表
3. `<TAB>` 在候选列表中切换选项

## 用来构建 Ex 命令的地址和范围的符号

| 符号       | 地址                         |
|:-----------|:-----------------------------|
| `{number}` | 行号                         |
| `1`        | 文件的首行                   |
| `$ `       | 文件的末行                   |
| `0`        | 虚拟行，位于文件首行上方     |
| `.`        | 光标所在行                   |
| `'m`       | 包含位置标记 m 的行          |
| `'<`       | 最近一次高亮选区的首行       |
| `'>`       | 最近一次高亮选区的末行       |
| `%`        | 整个文件（`:1,$` 的简写形式) |

## 其他

- `<C-r><C-w>` 将光标下单词插入命令行中
- `<C-r><C-a>` 将光标下字串插入命令行中
- `q/` 打开查找命令历史的命令行窗口
- `q:` 打开 Ex 命令历史的命令行窗口
- `<C-f>` 从命令行模式切换到命令行窗口
- `<up>` `<down>` 向前/向后遍历历史命令。若已输入部分内容，则只遍历已输入内容开头的命令。
- `<C-p>` `<C-n>` 向前/向后遍历历史命令。不会根据已输入内容进行过滤。

# 插入特殊字符

在插入模式中输入 `<C-v>{code}`。Vim 接受的字符编码共包含 3 位数字，不满 3 位的在开头补 0，超过 3 位的用 `<C-v>u{code}` 来输入，其中 {code} 为十六进制编码。

`ga` 命令查看光标所在字符的编码。

如果 `<C-v>` 后面跟一个非数字键，则会插入这个按键本身所代表的字符。比如若开启了'expandtab'选项，则 {Tab} 键会插入空格，而 `<C-v>{Tab}` 会插入制表符。

- `<C-v>{123}` 以十进制字符编码插入字符
- `<C-v>u{1234}` 以十六进制字符编码插入字符
- `<C-v>{nodigit}` 按原义插入非数字字符
- `<C-k>{char1}{char2}` 插入以二合字母表示的字符

- `:h digraphs-default` 缺省的二合字母集依从的惯例
- `:h digraph-table` 二合字母列表

# 窗口

**新建窗口**

- `<C-w>s`：水平切分当前窗口，新窗口仍显示当前缓冲区
- `<C-w>v`：垂直切分当前窗口，新窗口仍显示当前缓冲区
- `:sp[lit] {file}`：水平切分当前窗口，并在新窗口中载入{file}
- `:vsp[lit] {file}`：垂直切分当前窗口，并在新窗口中载入{file}

**在窗口间切换，详见 `:h window-move-curse`**

- `<C-w>w` `<C-w><C-w>` 在窗口间循环切换
- `<C-w>h` 切换到左边的窗口
- `<C-w>j` 切换到下边的窗口
- `<C-w>k` 切换到上边的窗口
- `<C-w>l` 切换到右边的窗口

**关闭窗口**

- `:clo[se]` `<C-w>c`：关闭活动窗口
- `:on[ly]` `<C-w>o`：只保留活动窗口，关闭其他所有窗口

**改变窗口大小及重新排列窗口，详见 `:h window-resize`**

- `<C-w>=`：使所有窗口等宽、等高
- `<C-w>_`：最大化活动窗口的高度
- `<C-w>\|`：最大化活动窗口的宽度
- `[N]<C-w>_`：把活动窗口的高度设为[N]行
- `[N]<C-w>\|`：把活动窗口的宽度设为[N]列

**重排窗口，详见 `:h window-moving`**

- `<C-w>r` `<C-w><C-r>`：将所有窗口下移/右移，只影响当前窗口所在列/行
- `<C-w>R`：将所有窗口上移/左移，只影响当前窗口所在列/行
- `[N]<C-w>x` `[N]<C-w><C-x>`：将当前窗口与下一个窗口交换，若无下个窗口，则与上一个窗口交换，光标会切换到目标窗口。通过指定[N]来将当前窗口与第[N]个窗口交换（第一个窗口为1），只影响当前窗口所在行/列
`<C-w>K`：将当前窗口移到最上方，并独占最大宽度
`<C-w>J`：将当前窗口移到最下方，并独占最大宽度
`<C-w>H`：将当前窗口移到最左方，并独占最大高度
`<C-w>L`：将当前窗口移到最右方，并独占最大高度
`[N]<C-w>T`：将当前窗口移到新标签页。若指定[N]则新标签页会置于第N个标签页之前，否则置于当前标签页之后。

**其他**

- `:lcd {path}`：设置当前窗口的本地工作目录
- `:windo lcd {path}`：设置当前标签页中所有窗口的本地工作目录

# 标签页

**打开及关闭标签页**
`:tabe[dit] {filename}`：在新标签页中打开{filename}。若省略{filename}，则新标签页会包含空缓冲区
`:tabc[lose]`：关闭当前标签页及其中所有窗口
`:tabo[nly]`：只保留活动标签页，关闭所有其他标签页
`<C-w>T`：若当前标签页不止一个窗口，则把当前窗口移到一个新标签页中

**在标签页间切换**
`:tabn[ext] {N}` `{N}gt`：跳到标签页{N}，标签页编号从1开始。省略{N}则跳到下一个标签页
`:tabp[revious]` `{N}gT`：同上，方向相反

**重排标签页**
`:tabmove {N}`：当N为0时，当前标签页会被移到开头；若省略了N，则移到末尾；也可鼠标拖动

# quickfix

- `:cope[n] [n]` 打开 quickfix 窗口，可指定高度。
- `:ccl[ose]` 关闭 quickfix 窗口。
- `:cl[ist] [{args}]` 列出所有项目。
  - `{from} [, {to}]` 指定范围。
  - `+{count}` 列出当前项目和后面 `{count}` 个项目。
- `:cc [N]` 跳转到第 N 项。
- `:cp[revious]` 上一项。
- `:cn[ext]` 下一项。
- `:cr[ewind]|cfir[st] [n]` 跳转到第 n 项，若省略 n 则跳转到第一项。
- `:cla[st] [n]` 跳转到第 n 项，若省略 n 则跳转到最后一项。
- `:cold[er]` 切到前一个列表。
- `:cnew` 切到后一个列表。

# 打开文件

`:pwd`：打印工作目录

**相对于当前工作目录打开一个文件**
`:e[dit] {file}`：{file} 可以是相对于工作目录的文件路径，也可以是绝对路径
`:e[dit] %:h/{file}`：`%:h`表示当前活动文件的目录路径，也可使用 `:e %:h<Tab>` 显示出路径后再继续输入文件名
在 _vimrc 文件中加入 `cnoremap <expr> %% getcmdtype()==':' ? expand('%:h').'/' : '%%'`

**通过文件名查找文件 详见 `:h file-searching`**
`set path+={root}/**`：首先要设置path变量。{root}为某个目录路径，**通配符表示其下所有子目录
`:find {file}`：在 path 中查找文件

**打开文件管理器 使用 netrw [API](http://vimcdoc.sourceforge.net/doc/pi_netrw.html)**
`:e[dit] {path}`：打开文件管理器，{path}是目录名而不是文件名
`:e[dit].`：打开工作目录
`:e[dit] %:h`：打开当前活动缓冲区所在目录
`:E[xplore]`：打开当前活动缓冲区所在目录或指定目录
`:Se[xplore]`：在一个水平切分窗口里打开当前活动缓冲区所在目录或指定目录
`:Ve[xplore]`：在一个垂直切分窗口里打开当前活动缓冲区所在目录或指定目录
`:Te[xplore]`：在一个新标签页中打开当前目录或指定目录
`<C-^>`：在文件管理器中切换到上一个打开的文件/上一个进入的目录
`%`：创建一个新文件
`d`：创建一个新目录
`D`：删除文件/目录
`-`：进入上一层目录，不能返回根目录
`a`：在三种方式间切换：正常显示、隐藏（不显示匹配）、显示（只显示匹配）
`c`：使浏览中的目录成为当前目录
`i`：切换列表方式
`<C-l>`：刷新目录列表
`o`：打开新浏览窗口，进入光标所在目录，使用水平分割
`v`：打开新浏览窗口，进入光标所在目录，使用垂直分割
`p`：预览文件
`P`：在前次使用的窗口里浏览
`r`：反转排序
`R`：重命名
`s`：选择排序方式：按名字、时间或文件大小
`S`：指定按名字排序的后缀优先
`t`：在新标签页里打开光标所在的文件/目录
`mf`：标记文件/目录，用于后续批量操作
`x`：指定某个程序来打开文件

**把文件保存到不存在的目录中**
`:!mkdir -p %:h`：创建任何不存在的中间目录，然后`:w`

# 缓冲区列表

- 查看
  - `:ls[!]` `:buffers[!]`: 查看缓冲区列表，加上 `!` 将同时显示列表外缓冲区。
- 切换
  - `<C-6>`               : 在当前文件和轮换文件间切换。
  - `:bp[rev]` `:bn[ext]` : 反向/正向切换一个缓冲区。
  - `:bfirst` `:blast`    : 跳到缓冲区列表的开头/结尾。
  - `:b[uffer] {num/name}`: 打开指定编号/名字的缓冲区。
- 删除
  - `:bd[elete] {n1} {n2}`: 删除指定编号的若干个缓冲区，即置为“列表外缓冲区”。
  - `{n},{m} bd[elete]`   : 删除编号从 n 到 m 的所有缓冲区，即置为“列表外缓冲区”。
  - `:bw[ipeout]`         : 彻底删除缓冲区，用法同 `:bd[elete]`。
- 执行
  - `:bufdo`              : 对所有缓冲区执行命令。

**隐藏缓冲区**

切换缓冲区时（包括执行 `:bufdo` `:argdo` 时），若缓冲区尚未保存，则必须在命令后加上 `!`。可通过 `hidden` 选项来让被切换走的缓冲区变为隐藏缓冲区。

- `set [no]hid[den]` 针对全局
- `set bufhidden|bh` 针对缓冲区
- `:hide {cmd}` 针对命令

|符号|含义                                 |
|:---|:------------------------------------|
|u   |列表外缓冲区 unlisted-buffer         |
|%   |当前缓冲区                           |
|#   |轮换缓冲区                           |
|a   |激活缓冲区，缓冲区被加载且显示       |
|h   |隐藏缓冲区，缓冲区被加载但不显示     |
|=   |只读缓冲区                           |
|-   |不可改缓冲区，'modifiable' 选项不置位|
|+   |已修改缓冲区                         |


# 参数列表

- 查看
  - `:ar[gs]`：显示当前参数列表内容
- 编辑
  - `:ar[gs] {argList}`：设置参数列表内容，其中 {argsList} 可以包括文件名、通配符 [^wildcard] 和 shell 命令输出结果（`:ar {cmd}`）
  - `:arge[dit] [{name}]`：将 {name} 添加到参数列表并编辑该文件，缺省为当前缓冲区的文件名
  - `:arga[dd] [{name}]`：将 {name} 添加到参数列表，缺省为当前缓冲区的文件名
  - `:argd {pattern}` `:[range]argd`：按照模式删除参数；按照范围删除参数，参数列表从 1 开始计数，`$` 表示末尾，`%` 表示全部，`.` 或不指定表示当前
- 切换
  - `:[count]n[ext]`：编辑当前之后第 count 个文件
  - `:[count]N[ext]` `:[count]prev[ious]`：编辑当前之前第 count 个文件
  - `:rew[ind]` `:fir[st]`：编辑第一个文件
  - `:la[st]`：编辑最后一个文件
- 执行
  - `:[range]argdo {cmd}`：对参数列表中的文件批量执行命令；要先设置 `hidden` 选项。

[^wildcard]: `*` 匹配 0 个或多个字符，范围仅限指定的目录，不含其子目录（如：`*.js`）；`**` 也匹配 0 个或多个字符，范围包括指定目录及其子目录（如：`**/*.js`）。

# 在文件中移动

动作命令详见 `:h motion.txt`

## 滚屏

- `<C-u>` 向上滚动半屏
- `<C-d>` 向下滚动半屏
- `<C-b>` 向上滚动一屏
- `<C-f>` 向下滚动一屏
- `[{n}]<C-y>` 向上滚动 {n} 行，缺省 1 行
- `[{n}]<C-e>` 向下滚动 {n} 行，缺省 1 行

## 在实际行和屏幕行间移动

- `j`  向下移动一个实际行
- `gj` 向下移动一个屏幕行
- `k`  向上移动一个实际行
- `gk` 向上移动一个屏幕行
- `0`  移动到实际行的行首
- `g0` 移动到屏幕行的行首
- `^`  移动到实际行的第一个非空白字符
- `g^` 移动到屏幕行的第一个非空白字符
- `$`  移动到实际行的行尾
- `g$` 移动到屏幕行的行尾

## 基于单词、字串、句子、段落移动

**基于单词移动**

`w`  移动到下一个词头
`b`  移动到上一个词头
`e`  移动到下一个词尾
`ge` 移动到上一个词尾

**基于字串移动**

只需将上述4个命令中的`w` `b` `e`换成大写

单词由字母、数字、下划线组成，或由连续的其他符号组成；字串由非空白符组成，由空白符分隔。

**基于句子移动**

`(` `)` 跳转到上一句/下一句开头

**基于段落移动**

`{` `}` 跳转到上一段/下一段开头

## 行内查找

- `f{char}` 正向移动到下一个{char}所在之处
- `F{char}` 反向移动到上一个{char}所在之处
- `t{char}` 正向移动到下一个{char}所在之处的前一个字符上
- `T{char}` 反向移动到上一个{char}所在之处的后一个字符上
- `;` 正向重复上一次字符查找命令
- `,` 反向重复上一次字符查找命令

## 文本对象

`i` 表示分隔符内的文本，`a` 表示包含分隔符本身及其内部文本。用 surround 插件可以方便地操作分隔符。

- `i)` `ib` 圆括号(parentheses)；`i)`与`i(`一样，下同
- `i}` `iB` 花括号(braces)
- `i]` 方括号(brackets)
- `i>` 尖括号(angle brackets)
- `i'` 单引号(single quotes)
- `i"` 双引号(double quotes)
- ``i` `` 反引号(backticks)
- `it` XML标签(tags)

- `iw` `aw` 当前单词；当前单词及一个空格
- `iW` `aW` 当前字串；当前字串及一个空格
- `is` `as` 当前句子；当前句子及一个空格
- `ip` `ap` 当前段落；当前段落及一个空行

## 标记

- `m{a-zA-Z}` 设置标记。小写只在该缓冲区内可见，大写全局可见
- ``{mark}` 跳转到标记处的行和列
- `'{mark}` 跳转到标记处的行首
- `:marks` 获取所有标记的列表
- `:delmarks {mark1} {mark2}` 删除指定标记；同时删除的多个标记之间用空格分隔

**自动位置标记**

- `` `  `` 当前文件中上次跳转动作之前的位置
- `` `. `` 上次修改的地方
- `` `^ `` 上次插入的地方
- `` `[ `` 上次修改或复制的起始位置
- `` `] `` 上次修改或复制的结束位置
- `` `< `` 上次高亮选区的起始位置
- `` `> `` 上次高亮选区的结束位置
- `%` 在一组开、闭括号间跳转，vim 会在跳转发生的地方设置一个标记

# 在文件间跳转

哪些动作算**跳转**？

- 简单来说，大范围的动作命令可能被当作跳转，小范围的动作命令只能算作移动；
- 面向句子、段落，甚至文件之间的移动（如 `:edit`）都算跳转；

以下是部分跳转命令

- `[count]G` 跳转到指定的行号
- `/pattern<CR>` `?pattern<CR>` `n` `N` 跳转到模式匹配处
- `%` 跳转到括号匹配处
- `(` `)` 跳转到上一句/下一句开头
- `{` `}` 跳转到上一段/下一段开头
- `H` `M` `L` 跳转到屏幕最上方/正中间/最下方
- `gf` 跳转到光标下的文件名
- `<C-]>` 跳转到光标下关键字的定义处
- `'{mark}` `` `{mark} `` 跳转到一个位置标记

## Vim 为每个窗口维护了一份独立的跳转列表

- `:ju[mps]` 查看跳转列表
- `:cle[arjumps]` 清除跳转列表
- `<C-o>`/`<C-i>` 反向/正向遍历跳转列表

## Vim 为每个缓冲区维护了一份独立的改变列表

- `:changes` 查看变列表
- `g;`/`g,` 反向/正向遍历改变列表；不会被记录进跳转列表
- `u<C-r>` 返回上次修改处。这是一种取巧的做法，通过“撤销->重做”来移动光标

## Vim 会自动创建一些位置标记

`gi` 返回上次退出 insert 模式的位置，并进入 insert 模式

## Vim 会把文档中的文件名当成一个超链接

- `gf` 打开光标下的文件名，此动作会在跳转列表中增加一条记录
- `:set suffixesadd+=.suffix` 指定一个或多个文件扩展名，当 Vim 用 `gf` 搜索文件名时，会尝试用这些扩展名
- `:set path+={path}`  `gf` 除了会在工作目录的相对目录中搜索外，也会在 `path` 中的路径下搜索

# 寄存器

- 无名寄存器
  - `""`：缓存最后一次操作内容
- 具名寄存器
  - `"a`~`"z`：通过指定名称来使用。同一字母大小写是同一个寄存器，小写会覆盖寄存器内原有内容，大写会续接原有内容。
- 数字寄存器
  - `"0`：复制专用寄存器，只受 `y{motion}` 影响
  - `"1`~`"9`：缓存最近 9 次删除内容
- 行内删除寄存器
  - `"-`：行内删除寄存器
- 只读寄存器
  - `"%`：当前文件名
  - `"#`：轮换文件名（当前交替文件名）
  - `".`：上次插入的文本
  - `":`：上次执行的 Ex 命令
- 模式寄存器
  - `"/`：上次查找的模式
- 表达式寄存器
  - `"=`：只读，用于执行表达式命令
- 黑洞寄存器
  - `"_`：不缓存内容，用于彻底删除
- 系统剪贴板
  - `"+`：系统剪贴板
  - `"*`：选择专用寄存器。在X11视窗系统中代表主剪贴板（primary），在其余系统中同系统剪贴板
  - `"~`：拖放操作寄存器

**引用一个寄存器**

- insert mode：`<C-r>{reg without "}`。无需输入寄存器名称中的 `"`。
- normal mode：`x`、`s`、`d{motion}`、`c{motion}`、`y{motion}` 以及它们对应的大写命令，在命令前加 `{reg}` 前缀指定引用某个寄存器。
- visual mode：选中高亮内容后同 normal mode。
- Ex mode：`:delete {reg}`、`:yank {reg}`、`:put {reg}`。
- 脚本：`@{reg without "}`

缺省引用的是无名寄存器 `""`。

**查看寄存器内容**

`:reg[s] [{reg}]` `:di[splay] [{reg}]`：查看寄存器，若不指定寄存器名字，则查看全部。

**粘贴**

- `p`：粘贴至光标后
- `P`：粘贴至光标前
- `gp`：粘贴至当前行之前，光标落于被粘贴行末尾
- `gP`：粘贴至当前行之后，光标落于被粘贴行开头

在终端中使用系统剪贴板进行粘贴时可能会遇到一些问题，详见《Vim实用技巧》之《技巧63》。

# 宏

**录制宏**

1. `q{reg}`：开始录制，宏存放于 {reg} 指定的寄存器中，若 {reg} 为大写的具名寄存器，则会将新宏续接在原宏后面
2. `q`：结束录制

**执行宏**

- `[n]@{reg}`：执行 {reg} 中保存的宏，可指定运行次数
- `@@`：重复上次调用的宏

*以上对寄存器的引用无需使用 `"`*

**编辑宏**

以下以寄存器 a 为例。

1. 粘贴宏内容：`:put a` 粘贴至当前行下方，或者 `"ap` 粘贴至光标之后。
2. 编辑宏内容。
3. 保存新内容至寄存器：`0`，`"ay$`。

# 模式

## 模式

`:h perl-patterns`

**全局设置大小写敏感性**

- `set ignorecase|ic` `set noignorecase|noic`: 大小写敏感关/开（会影响自动补全）。
- `set smartcase|scs` `set nosmartcase|noscs`: 智能模式，只有在设置了 `ignorecase` 时才生效。

**临时设置大小写敏感性**

- `\c` `\C`:忽略/区分大小写，可出现在模式的任意位置

**4种语法模式**

- `\v`:very magic 模式，除了 `_`、大小写字母、数字外，都具有特殊含义，`#` 暂时无特殊含义，但是 Vim 保留扩展其特殊含义的可能
- `\V`:very nomagic 模式，除了 `\`、`/`、`?` 外都无特殊含义（后 2 个为查找域结束符）
- `\m`:magic 模式，缺省模式，除了 `*`、`^`、`$`、`.`、`~`、`[`、`]` 外都无特殊含义
- `\M`:nomagic 模式，除 了 `^`、`$` 外都无特殊含义

**模式项 详见`:h pattern-overview`**

- `/(expr1)@>expr2`:固化分组。详见另一片正则表达式的博文。
- `/expr1(expr2)@=`:肯定型顺序环视。匹配所有满足后置表达式为expr2的expr1。
- `/expr1(expr2)@!`:否定型顺序环视。匹配所有满足后置表达式**不**为expr2的expr1。
- `/(expr1)@<=expr2`:肯定型逆序环视。匹配所有满足前置表达式为expr1的expr2。
- `/(expr1)@<!expr2`:否定型逆序环视。匹配所有满足前置表达式**不**为expr1的expr2。
- `\zs`:肯定型逆序环视
- `\ze`:肯定型顺序环视

**不捕获分组内容**

- `%(pattern)`：不捕获分组内容

**限制符**

- `{n,m}`:  尽可能多地匹配 n ~ m 个元素。
- `{n}`:    尽可能多地匹配 n 个元素。
- `{n,}`:   尽可能多地匹配 n 及更多个元素。
- `{,m}`:   尽可能多地匹配 0 ~ m 个元素。
- `{}`:     尽可能多地匹配 0 及更多个元素。
- `{-n,m}`: 尽可能少地匹配 n ~ m 个元素。
- `{-n}`:   尽可能少地匹配 n 个元素。
- `{-n,}`:  尽可能少地匹配 n 及更多个元素。
- `{-,m}`:  尽可能少地匹配 0 ~ m 个元素。
- `{-}`:    尽可能少地匹配 0 及更多个元素。


## 查找

- `set wrapscan|ws`/`set nowrapscan|nows` 打开/关闭循环查找
- `set hlsearch|hls`/`set nohlsearch|nohls|hls!` 打开/关闭匹配高亮
- `:noh` 关闭高亮，直到下一次查找
- `set incsearch|is`/`set noincsearch|nois` 打开/关闭增量查找
- `<C-r><C-w>` 用当前预览的匹配结果对查找域进行自动补全
- `:%s/{pattern}//gn` 统计匹配数量。{pattern} 可省略，即使用当前查找模式；`n` 表示抑制替换
- `q/` 调出查找历史命令行窗口

## 替换

完整的substitute语法：
`:[range]s[ubstitute]/{pattern}/{string}/[flags]`

标志位 *`:h s_flags`*：

- `g`：在全局范围内执行，即修改一行内所有匹配。
- `c`：让用户确认或拒绝每一处修改。*`:h :s_c`*
    选项：
    - `y`：替换此处匹配。
    - `n`：忽略此处匹配。
    - `q`：退出替换过程。
    - `l`：“last”——替换此处匹配后退出。
    - `a`：“all”——替换此处与之后所有的匹配。
    - `<C-e>`：向下滚动屏幕，即文本向上移动。
    - `<C-y>`：向上滚动屏幕，即文本向下移动。
- `n`：抑制替换行为，转而报告匹配个数。
- `e`：屏蔽错误信息。
- `&`：重用上一次 `substitute` 命令所用的标志位，必须写在 flags 的第一位。
- `i`：忽略查找域的大小写。
- `I`：不忽略查找域的大小写。
- `p`：显示最后一处替换所在行。
- `#`：同 `p`，并加上行号。
- `l`：同 `p`，但显示效果同 `:list`，即用 `^` 显示无法打印的字符，并在行尾添加 `$`。
- `r`：只对不带参数的 `:&` 和 `:s` 有效，作用同 `:~`：用最近一次的查找或替换命令中的模式来作为查找域，若最近一次的是替换命令且它的查找域为空，则继续在历史记录中向前查找。

替换域中的特殊字符（部分）
详见*`:h sub-replace-special`*
|符号|描述|
|:--|:--|
|`\r`|换行符|
|`\t`|制表符|
|`\\`|反斜杠|
|`\n`|第n个子匹配|
|`\0`|匹配模式的所有内容|
|`&`|匹配模式的所有内容|
|`~`|使用上一次调用 `:substitute` 时的替换域|
|`\={Vim Script}`|执行{Vim Script}表达式，并将结果作为替换域|

替换相关命令
|命令|用途|
|:--|:--|
|`:[range]s[ubstitute] [flags] [count]` `:[range]& [flags] [count]`|重复上一次替换操作，但忽略上次的flags，可手动添加flags|
|`:[range]&& [count]`|重复上一次替换操作，并带有上次的flags|
|`:[range]~[&][flags] [count]`|同 `:&r`|
|`&`|同 `:s`|
|`g&`|同 `:%s//~/&`，其中查找域的内容用的是最近一次查找或替换命令所用的查找域*global substitute*|
|`:[range]sno[magic]`|同`:substitute`，但使用nomagic模式|
|`:[range]sm[agic]`|同`:substitute`，但使用magic模式|

global命令

|命令|用途|
|:--|:--|
|`:[range]g[lobal]/{pattern}/[.,{finish}] [cmd]`|在range（缺省为整个文件`%`）范围内，对{pattern}匹配处执行cmd命令（缺省为`:p[rint]`），可以指定cmd的执行范围，`.`表示{pattern}匹配行，{finish}可通过偏移或匹配命令`/{pattern}/`指定|
|`:[range]g[lobal]!/{pattern}/[.,{finish}] [cmd]`|在range（缺省为整个文件`%`）范围内，对{pattern}不匹配处执行cmd命令（缺省为`:p[rint]`）|
|`:[range]v[global]/{pattern}/[.,{finish}] [cmd]`|同上|


# 工具

## vim-surroud

`ds` 和 `cs` 命令接受一个 target 作为第一个参数。目前所有的 target 都是单字符的。

- `(` `)` `[` `]` `{` `}` `<` `>` 代表它们自身以及与其对应的另一个字符。
- `b` `B` `r` `a` 分别是 `)` `}` `]` `>` 的别名，其中前两个沿用 Vim 的设定。
- `'` `"` ``` `` 代表它们自身（成对地），它们仅会在当前行被搜索。
- `t` 代表一对 HTML/XML 标签。
- `w` `W` `s` `p` 分别代表一个单词、字串、句子、段落。

`cs` `ys` `vS` 需要一个单字符的 replacement 参数

- 若使用 `)` `]` `}` `>`，则会正确匹配另一半符号。
- 若使用 `(` `[` `{`（`<` 除外），则会正确匹配另一半符号，并在中间填充额外的空格。
- `b` `B` `r` `a` 分别是 `)` `}` `]` `>` 的别名。
- `<C-]>` 代表了 C 语言风格的 `{}`。
- 若使用 `t` `<`，则 Vim 会要求继续输入标签的属性。可以输入 `<CR>` 或 `>` 来表示完成输入。Vim 会自动匹配闭标签。若使用 `<C-t>`，则生成的标签会自动换行。
- 若使用 `s` ，则 Vim 会自动在开头加个空格，这样可以有效地删除 `()`。
- surround 默认将字母 `l` 作为 Latex 的分隔符。

除了上述的字符外，其余单字符会与其自身匹配成一对。

**normal mode**

- `ds<s>`：删除分隔符 `<s>`
- `cs<s><t>`：将 `<s>` 替换为 `<t>`
- `cS<s><t>`：将 `<s>` 替换为 `<t>`，并添加换行和缩进
- `ys{motion}<s>` `ys{文本对象}<s>`：在指定文本外添加分隔符
- `yS{motion}<s>` `yS{文本对象}<s>`：在指定文本外添加分隔符，并添加换行和缩进
- `yss<s>`：在当前行外添加分隔符
- `ySs<s>` `ySS<s>`：在当前行外添加分隔符，并添加换行和缩进

**visual mode**

- `S<s>`：在高亮选区外添加分隔符

**insert mode**

- `<C-g>s<s>` `<C-S><s>`：插入分隔符并将光标置入分隔符中间；后者在终端中会使终端冻结

**个性化**

*`:h surround-customizing`*

1. 在特定类型文件中启用自定义的分隔符。
  `autocmd FileType php let b:surround_45 = "<?php \r ?>"`
  其中，
  - `php` 是指定的文件类型；
  - `b` 表示后面的变量作用域为当前缓冲区；
  - `surround_` 为变量名中的固定部分；
  - `45` 为字符 `-` 的 ASCII 码（字符和 ASCII 码之间的转换可用 `char2nr()` 和 `nr2char()` 函数）；
  - 回车符 `\r` 会被原始文本（即被分隔符包围的原文）替换。
2. 动态输入分隔符标签中的内容。
  `let g:surround_108 = "\\begin{\1environment: \1}\r\\end{\1\1}"`
  - `\1` 表示输入参数，最高支持到 `\7`；
  - 两个 `\1` 之间的内容会作为提示在用户输入时弹出，并最终会被输入的内容替换；
  - 回车符 `\r` 会被原始文本（即被分隔符包围的原文）替换。
3. 使用正则表达式完成额外的功能。
    `let g:surround_108 = "\\begin{\1environment: \1}\r\\end{\1\r}.*\r\1}"`
    - 在第二组 `\1` 中，第一个 `\r` 后面跟的是替换操作所用的 pattern，第二个 `\r` 后面跟的是替换操作所用的 replacement。最终的效果是：第一组 `\1` 之间的内容会被输入内容替换；第二组 `\1` 之间的内容先被输入内容替换，然后用 pattern 进行匹配，匹配到的内容会被 replacement 替换，至此确定最终结果。
    `let g:surround_{char2nr("d")} = "<div\1id: \r..*\r id=\"&\"\1>\r</div>"`
    - 其中 `&` 表示在第一组 `\r` 中匹配到的内容（`:h s/\&`）
4. 为分隔符自动添加后缀
    `let g:surround_insert_tail = "<++>"`
    - 在 insert 模式下使用 `<C-g>s<s>` 插入分隔符时会自动给分隔符添加后缀；
    - 该变量的名称固定，且作用域必须是全局 `g:`；
    - `<++>` 为自定义的后缀；
    - 只能在 insert 模式下生效。

## Vim 内部的 Grep

`:vim[grep][!] /{pattern}/[g][j] {file}...`
- `g`：缺省只为每个出现匹配的行创建一条记录。此参数的作用：若同一行中有多处匹配，则为每一处匹配创建一条记录。
- `j`：缺省 Vim 会跳转到第一处匹配。此参数的作用：只更新 quickfix 列表，但不跳到第一处匹配。
- `file`：可接受的参数同`:args`。

## ctrlp

### 快捷键

`:h ctrlp-commands`

- `<C-p>` `:CtrlP [starting-dir]` 启动文件查找模式，可指定目录。
- 在输入框中
  - `<C-d>` 在全路径搜索(`>>>`)和文件名搜索(`>d>`)之间切换。
  - `<C-r>` 在字符串匹配(`>>>`)和正则匹配(`r>>`)之间切换。
  - `<C-f>` `<C-up>` `<C-b>` `<C-down>` 循环切换搜索模式：普通（文件查找）、缓冲区查找、最近历史查找（MRU）。
  - `<tab>` 自动补全目录名。
  - `<S-tab>` 在匹配窗口和输入框之间切换。
    - 在匹配窗口按下任意字符来选中首字符匹配的项目。多次按同一字符会在多个首字符匹配的项目间循环。
- 移动
  - `<C-k>` `<up>` `<C-j>` `<down>` 向上、向下。
  - `<C-a>` `<C-e>` 移至输入框开头/末尾。
  - `<C-h>` `<left>` `<C-^>` `<C-l>` `<right>` 左移、右移。
- 编辑
  - `<C-]>` `<bs>` `<del>` 左删、右删一个字符。
  - `<C-w>` 左删一个词。
  - `<C-u>` 清空。
- 浏览输入记录
  - `<C-n>` `<C-p>` 上翻、下翻。
- 打开/创建文件
  - `<CR>` 在当前窗口打开。
  - `<C-t>` 在新 tab 打开。
  - `<C-v>` 在新水平分割窗口打开。
  - `<C-x>` `<C-x>` `<C-CR>` `<C-s>` 在新垂直分割窗口打开。
  - `<C-y>` 创建新文件。
- 打开多个文件
  - `<C-z>` 标记/取消标记一个待打开的文件；标记/取消标记一个文件，它所在的目录会作为创建新文件的目录 `<C-y>`。
  - `<C-o>` 打开所有被标记的文件。若没有标记文件的话就根据以下选项打开一个控制台对话框：
    - `t` 在 tab 中。
    - `v` 在垂直分割窗口中。
    - `h` 在水平分割窗口中。
    - `r` 在当前窗口。
    - `i` 作为隐藏缓冲区。
    - `x` （可选）使用 `g:ctrlp_open_func` 定义的函数。
    - `a` 标记匹配窗口中的所有文件。
    - `d` 将 ctrlp 的工作目录切换到所选文件的目录。
- 函数快捷键
  - `<F5>` 全量刷新匹配窗口和最近历史记录（MRU）。
  - `<F7>` 清空 MRU 或删除 MRU 中被标记的项目。
- 粘贴
  - `<Insert>` `<MiddleMouse>` 将剪贴板粘贴至输入框。
  - `<C-\>` 弹出一个对话框来选择粘贴的内容：光标下的词、光标下的文件名、搜索寄存器内容、最近的框选内容、系统剪贴板、指定寄存器。

### 输入格式

- 简单字符串：'abc' 相当于 `a[^a]\{-}b[^b]\{-}c`。
- 正则表达式：按照 vim 的正则规则。
- 字符串后紧跟 `:{cmd}`：指定打开文件后执行的命令。
- 切换目录
  - `..`：往上一级。后续每多一个点表示再往上一级。
  - `@cd {path}` 切换目录。
  - `@cd %:h` 切换到当前文件的目录。
  - `/` `\` 切换到项目根目录。
- `?` 帮助文件。

### 自定义

`:h ctrlp-customization` `:h ctrlp-options`

- 更改启动命令
  - `let g:ctrlp_map = '<C-p>'`
  - `let g:ctrlp_cmd = 'CtrlP'`
- 不带指定目录启动时
  - `let g:ctrlp_working_path_mode = 'ra'`
    - `c` 当前文件所在目录。
    - `a` 当前文件所在目录，除非它是 `cwd` 的子目录。
    - `r` 父目录中最近一个包含版本控制信息的目录：`.git`、`.hg`、`.svn`、`.bzr`、`_darcs`。
      - `let g:ctrlp_root_marks = ['pom.xml', '.p4ignore']` 自定义父目录的标识。
    - `w` 作为 `r` 的修饰符，从 `cwd` 开始搜索，而不是当前文件所在目录。
    - `0` `''` 禁用此特性。
- 排除文件和目录

  ```
  " vim 自带的排除方式
  set wildignore+=*/tmp/*,*.so,*.swp,*.zip     " MacOSX/Linux
  set wildignore+=*\\tmp\\*,*.swp,*.zip,*.exe  " Windows

  " ctrlp 的排除方式
  let g:ctrlp_custom_ignore = '\v[\/]\.(git|hg|svn)$'
  let g:ctrlp_custom_ignore = {
    \ 'dir':  '\v[\/]\.(git|hg|svn)$',
    \ 'file': '\v\.(exe|so|dll)$',
    \ 'link': 'some_bad_symbolic_links',
    \ }
  ```

- 排除 `.gitignore` 中的文件和目录

  ```
  let g:ctrlp_user_command = ['.git', 'cd %s && git ls-files -co --exclude-standard']
  ```

- 自定义的搜索规则

  ```
  let g:ctrlp_user_command = 'find %s -type f'        " MacOSX/Linux
  let g:ctrlp_user_command = 'dir %s /-n /b /s /a-d'  " Windows
  ```

## fugitive

### 对象 fugitive-objects

- `:` 同 `:Gstatus`
- `.git/config` 仓库的配置文件
- `HEAD` .git/HEAD
- `refs/heads/x` .git/refs/heads/x
- 当前文件
  - `:%` 当前文件在暂存区中的对应文件
  - `@~2:%` 当前文件在 HEAD 的祖父提交中的对应文件
  - `:1:%` 冲突状态中，当前文件在共同祖先中的对应文件
  - `:2:#` 冲突状态中，当前文件在目标分支中的对应文件
- 指定文件名的文件
  - `./master` 工作目录下名为 master 的文件
  - `Makefile` 工作区中名为 Makefile 的文件
  - `@^:Makefile` HEAD 的父提交中名为 Makefile 的文件
  - `:Makefile` 暂存区中的 Makefile 文件
  - `!:Makefile` 包含当前文件的提交中的名为 Makefile 的文件
- 提交
  - `@` HEAD 指向的提交
  - `master^` master 的父提交
  - `master:` master 指向的 tree 对象
  - `!` 包含当前文件的提交
- 指定缓冲区中的文件
  - `:3:#5` 冲突状态中，缓冲区 #5 中文件在被合并分支中对应的文件
  - `!3^2` 包含缓冲区 #3 中文件的提交的第二父提交

### 命令

**通用**

- `Git {args}` 执行 Git 命令，相当于 `:!git {args}`。
- `Git! {args}` 执行 Git 命令，将结果输出到临时文件。
- `Gcd|Glcd [{dir}]` cd/lcd 到项目根目录。

**本地仓库**

- `G[status]` 在新窗口中查看 git status。
  - `g?` 查看 fugitive-mappings。
  - `-` add/reset 某个文件的修改。
  - `=` 展开各个差异块。
- `Gdiff` 查看当前文件工作区到暂存区的差异。
  - `Gsdiff` 横向分割窗口
  - `Gvdiff` 纵向分割窗口
- `Gcommit {args}` 提交当前文件，在新窗口中编辑提交信息。
- `Gmerge {args}` 合并分支，在 quickfix 窗口中列出冲突文件。
- `Grebase`
- `Gmove` git mv 同时会重命名缓冲区。
- `Gdelete` git rm 同时会删除缓冲区。
- `Gread` git checkout -- filename
- `Gwrite` 将缓冲区写到工作区和暂存区。
- `Ge` `Gsp` `Gvsp` `Gtabe` 查看指定的对象、树、提交或标签。

**远程仓库**

- `Gfetch {args}` `Gpull {args}` `Gpush {args}`

**历史日志**

- `Gblame [flags]` flags 会被传递给 git-blame。
  在 blame 窗口可以使用以下热键：
  - `g?` 帮助
  - `A` 重置尺寸至作者列
  - `C` 重置尺寸至提交列
  - `D` 重置尺寸至日期列
  - `q` 退出 blame
  - `gq` 退出 blame 并 `:Gedit` 工作区中的版本（不懂跟 `q` 有什么区别）
  - `<CR>` 打开光标下的提交
  - `o` 在水平分割窗口打开提交
  - `O` 在新标签页打开提交
  - `p` 在预览窗口打开提交
  - `-` 在光标下的提交上再做 blame
  - `~` 在第 [count] 个第一父提交上再做 blame
  - `P` 在第 [count] 父提交上再做 blame（相当于 HEAD^[count]）
- `0Glog` 通过 quickfix 窗口遍历当前文件历史。
- `Glog {args}` 通过 quickfix 窗口浏览提交历史。
  - `--` 查看所有提交，否则查看涉及当前文件的提交。
- `Gllog {args}` 类似 `Glog`，不过是用 location list。

### 热键

这些热键大多数都能在 `:Gstatus` 窗口和 fugitive 对象窗口使用。

**暂存和撤销**

- `s` stage 光标下的文件或块。
- `u` unstange 光标下的文件或块。
- `-` stage/unstage 光标下的文件或块。
- `<C-N>` 跳到下个文件或块。
- `<C-P>` 跳到上个文件或块。
- `X` 放弃光标下的改动，即 checkout 或 clean。
- `=` 在光标处切换内联差异比较。
- `<` 在光标处插入内联差异比较。
- `>` 在光标处移除内联差异比较。
- `i` 对于未跟踪文件，执行 `git add --intent-to-add`。否则移至下个块并展开内联差异比较
- `dd` 对光标下的文件执行 `:Gdiff`。
- `ds` 对光标下的文件执行 `:Gsdiff`。
- `dv` 对光标下的文件执行 `:Gvdiff`。
- `P` 对光标下的文件执行 `:Git add --patch` 或 `:Git reset --patch`。

**导航**

- `<CR>` 打开光标下的文件或对象。
- `o` `gO` 在新窗口中打开光标下的文件或对象。
- `O` 在新标签页中打开光标下的文件或对象。
- `~` 相当于 Git 的 `~`。
- `P` 相当于 Git 的 `^`。
- `C` 打开包含当前文件的提交。
- `<C-W>C` 在新窗口中打开包含当前文件的提交。

**提交**

- `cc` 创建提交。
- `ca` commit --amend 并编辑提交信息。
- `ce` commit --amend 且不编辑提交信息。
- `cw` 编辑最近一次提交。
- `cvc` 带 `-v` 提交。
- `cva` 带 `-v` amend。
- `cf` 为光标下的提交创建一个 fixup 提交。
- `cs` 为光标下的提交创建一个 squash 提交。
- `cA` 为光标下的提交创建一个 squash 提交并编辑信息。

**Rebase**

- `ri` 执行交互式 rebase，以光标下提交的父提交作为基准。
- `rf` 执行自动压缩 rebase，以光标下提交的父提交作为基准。
- `ru` 以 upstream 分支为基准执行交互式 rebase。
- `rp` 以 push 分支为基准执行交互式 rebase。

### 其他

`set statusline=%{FugitiveStatusline()}` 在状态栏添加分支信息

## gitgutter

更新频率取决于 vim 的 `updatetime` 选项，默认 4000 ms。

同一文件改动超过 500 处时，会隐藏标记。可设置 `let g:gitgutter_max_signs = 500`

### 快捷键

- `[c` `]c` 在改动块间跳转。
- `<leader>hp` 预览改动块。
- `<leader>hs` 暂存改动块。
- `<leader>hu` 回退改动块。（不是 unstage）
- 整体开关
  - `:GitGutterDisable`
  - `:GitGutterEnable`
  - `:GitGutterToggle`
- 缓冲区独立开关
  - `:GitGutterBufferDisable`
  - `:GitGutterBufferEnable`
  - `:GitGutterBufferToggle`
- 标记开关
  - `:GitGutterSignsDisable`
  - `:GitGutterSignsEnable`
  - `:GitGutterSignsToggle`
- 行高亮
  - `:GitGutterLineHighlightsDisable`
  - `:GitGutterLineHighlightsEnable`
  - `:GitGutterLineHighlightsToggle`

### 文本对象

- `ic` 改动块中的所有行。
- `ac` 改动块中的所有行以及后面紧跟的所有空行。

### 折叠

- `:GitGutterFold` 折叠/展开所有未改动的行。
- `zr` 展开改动块前后 3 行。

# 折叠

## 折叠相关的 Vim 变量

|变量          |含义                                    |
|:-------------|:---------------------------------------|
|`v:foldstart` |折叠首行的行号                          |
|`v:foldend`   |折叠末行的行号                          |
|`v:folddashes`|一个含有连字符的字符串，用来表示折叠级别|
|`v:foldlevel` |折叠级别                                |


## 折叠选项

- **颜色 COLORS**
关闭的折叠的颜色由 Folded 高亮组（*hl-Folded*）设定。折叠栏的颜色由 FoldColumn 组（*hl-FoldColumn*）设定。
`:highlight Folded guibg=grey guifg=blue`
`:highlight FoldColumn guibg=darkgrey guifg=white`
- **折叠级别 FOLDLEVEL**
包含行数小于等于 `foldlevel` 的折叠始终以展开的形式来显示。
`foldlevel` 被改变后立即生效。
- **折叠文本 FOLDTEXT**
`foldtext` 是一个字符串表达式，定义了关闭折叠时所显示的文字。如：
` :set foldtext=v:folddashes.substitute(getline(v:foldstart),'/\\*\\\|\\*/\\\|{{{\\d\\=','','g')`
在折叠时显示被折叠文本的首行的内容，并删除了其中的"/*"、"*/"、"{{{"。
以上功能也可通过一段函数来实现：
    ```
    set foldtext=MyFoldText()
    function MyFoldText()
      let line = getline(v:foldstart)
      let sub = substitute(line, '/\*\|\*/\|{{{\d\=', '', 'g')
      return v:folddashes . sub
    endfunction
    ```
- **折叠栏 FOLDCOLUMN**
折叠栏显示在窗口左侧，展示了所有折叠的结构。可以通过点击折叠栏中的“+”号来展开该折叠，通过点击任何其他非空字符来关闭该行所在折叠。
通过设定 `foldcolumn` 的值（0~12）来改变折叠栏的宽度。
- **其他选项**
`foldenable` `fen`：复位时打开所有折叠。
`foldexpr` `fde`：用于 `expr` 折叠模式的表达式。
`foldignore` `fdi`：用于 `indent` 折叠模式的字符。
`foldmarker` `fmr`：用于 `marker` 折叠模式的标志。
`foldmethod` `fdm`：当前折叠模式。
`foldminlines` `fml`：关闭折叠时的最小显示行数。
`foldnestmax` `fdn`：用于 `indent` 和 `syntax` 折叠模式的最大嵌套层数。
`foldopen` `fdo`：哪些命令可以打开已关闭的折叠。
`foldclose` `fcl`：非光标所在折叠是否关闭。缺省为""。若设置为“all”则所有高于 `foldlevel` 且非光标所在折叠都会自动关闭。

## 折叠方式

`set foldmethod = {method}` or `set fdm = {method}`

其中 {method} 有6个选项，各方式之间不兼容：

|命令  |含义                    |
|:-----|:-----------------------|
|manual|手工定义折叠            |
|indent|同等的缩进表示同级的折叠|
|expr  |用表达式来定义折叠      |
|syntax|用语法高亮来定义折叠    |
|diff  |对没有更改的文本进行折叠|
|marker|用特定标志标识折叠      |

当从 `manual` 模式切换到其他模式时，所有已存在的折叠会被删除并创建新折叠。而反过来则不会删除已存在的折叠。所以通常先选择 `manual` 之外的某种模式，得到自动生成的折叠，然后切换到 `manual` 模式。

- **manual**
  使用命令来创建折叠。折叠级别由嵌套层级决定。可以通过对某些行反复定义折叠来增加折叠级别。
  当退出编辑时，手工折叠会丢失。使用 `:mkview` 来保存折叠，使用 `:loadview` 来载入折叠。
- **indent**
  折叠级别由缩进除以 `shiftwidth` 并向下取整得到。嵌套的层数受 `foldnestmax` 限制。
  某些行会被忽略并得到相邻行的折叠级别（取较小值），这类行包括空行和以 `foldignore` 中某个字符开始的行。
- **expr**
  折叠级别通过对每行计算 `foldexpr` 的值来定义。该模式比较复杂，不常用。详见 `:h fold-expr`。
- **syntax**
  折叠由带有“fold”参数的语法项来定义（`:h syn-fold`）。折叠级别由折叠的嵌套层数决定。嵌套的折叠数量受 `foldnestmax` 限制。详见 `:h fold-syntax`。
- **diff**
  对未改动的文本或靠近改动的文本自动定义折叠。仅对当前窗口已设定 `diff` 选项来显示不同之处时有效，不然，整个缓冲区就是一个大折叠。详见 `:h fold-diff`。
- **marker**
  折叠根据 `foldmarker` 的值来定义，缺省值为“{{{,}}}”，建议不要修改。
  折叠行所显示的文本取决于 `foldtext` 选项，缺省为 foldtext() 的返回值，即折叠标志之前的文本。
  “3x{” 标志一个折叠的开始。可以用 “3x}” 标志一个折叠的结束（也可以不用，假如所有折叠都处于一个大嵌套中的话）。“3x{” 和 “3x}” 后都可以紧跟一个数字（0除外），表示该折叠的级别，数字越大，折叠所在的嵌套越深。
  带数字的标志和不带数字的标志可以混合使用。

## 创建和删除折叠

- `zf{motion}` `{visual}zf` `{count}zF` `:{range}fo[ld]`
  创建折叠，只在 `manual` 和 `marker` 模式下有效
- `zd` `{visual}zd`
  删除光标所在行的折叠，该折叠内的折叠将自动上移一级，或删除选中区域的所有折叠，且不可撤销。只在 `manual` 和 `marker` 模式下有效
- `zD` `{visual}zD`
  删除光标所在行及其内的所有折叠，或删除选中区域及其内的所有折叠，只在 `manual` 和 `marker` 模式下有效
- `zE`
  删除（eliminate）当前窗口内的所有折叠，只在 `manual` 和 `marker` 模式下有效

## 展开和关闭折叠

包含行数小于 `foldminlines` / `fdm` 的折叠将始终以展开的形式显示。`foldminlines`缺省为 1。

- `[count]zo`：打开光标下的折叠。当给定计数时，打开相应深度的折叠。在可视模式下，所选区域所有折叠都打开一级。
- `zO`：打开光标所在行及其内的所有折叠。在可视模式下，打开所选区域内的所有折叠。
- `[count]zc`：关闭光标下的折叠。当给定计数时，关闭从光标所在折叠开始的相应级数的折叠。在可视模式下，所选区域内的折叠被关闭一级。
- `zC`：关闭光标所在行及其内的所有折叠。在可视模式下，关闭所选区域内的所有折叠。
- `[count]za`：打开/关闭光标所在折叠。当给定计数时，打开/关闭相应数量的折叠。
- `zA`：打开/关闭光标所在折叠及其内的所有折叠。
- `zv`：查看（view）折叠，打开光标所在折叠及其上级的所有折叠。
- `zx`：更新折叠。撤销被手动打开和关闭的折叠：再次应用 `foldlevel`。然后使用 `zv`。同时强制重新计算折叠。使用 `foldexpr` 并且缓冲区发生改变但折叠不能正确地更新时，这会有用。
- `zX`：类似 `zx`。
- `zm`：折叠更多（more）：`foldlevel` 减1。
- `zM`：关闭所有折叠，将 `foldlevel` 设为0。
- `zr`：折叠减少（reduce）：`foldlevel` 加1。
- `zR`：打开所有折叠，将 `foldlevel` 设为最高级别。
- `:{range}foldo[pen][!]`：在 {range} 内打开一级折叠，若加上 `!` 则打开 {range} 内的所有折叠。
- `:{range}foldc[lose][!]`：在 {range} 内关闭一级折叠，若加上 `!` 则关闭 {range} 内的所有折叠。
- `zn`：不折叠（none）：重置 `foldenable`。所有折叠被打开。
- `zN`：正常折叠（normal）：设定 `foldenable`。所有折叠都表现为之前的样子。
- `zi`：反转 `foldenable` 的值。

## 在折叠间移动

`[count][z`：移动到当前打开折叠的开始。若已经在当前打开折叠的开始，则移动到上级折叠的开始。当给定计数时，重复[count]次`[z`命令。
`[count]]z`：移动到当前打开折叠的末尾。若已经在当前打开折叠的末尾，则移动到上级折叠的末尾。当给定计数时，重复[count]次`]z`命令。
`[count]zj`：移动到下一个折叠的开始，包括关闭的折叠。当给定计数时，重复[count]次`zj`命令。此命令可接在一个操作符（operator）后面。
`[count]zk`：移动到上一个折叠的末尾，包括关闭的折叠。当给定计数时，重复[count]次`zk`命令。此命令可接在一个操作符（operator）后面。

## 对折叠执行命令

`:[range]foldd[oopen] {cmd}`：对所有 [range] 范围内的、处于关闭折叠之外的行执行 {cmd}。执行过程类似 `:global` 命令：先标记所有需执行命令的行，然后执行命令。所以即使 {cmd} 会对折叠产生影响也不会改变 {cmd} 的执行范围。
`:[range]folddoc[losed] {cmd}`：对所有 [range] 范围内的、处于关闭折叠之内的行执行 {cmd}。类似上一个命令。

## 折叠行为

详见`:h fold-behavior`。


# 编码

- encoding（enc）：encoding 是 Vim 的内部使用编码，encoding 的设置会影响 Vim 内部的 Buffer、消息文字等。若载入的文件与 encoding 不同，则 Vim 会先转换成 encoding，然后在写入时再转回去。在 Unix 环境下，encoding 的默认设置等于 locale；Windows 环境下会和当前代码页相同。在中文 Windows 环境下 encoding 的默认设置是 cp936（GBK）。
- fileencodings（fencs）：Vim 在打开文件时会根据 fileencodings 选项来识别文件编码，fileencodings 可以同时设置多个编码，Vim 会根据设置的顺序来猜测所打开文件的编码。
- fileencoding（fenc）：Vim 在保存新建文件时会根据 fileencoding 的设置编码来保存。如果是打开已有文件，Vim 会根据打开文件时所识别的编码来保存，除非在保存时重新设置 fileencoding。
- termencodings（tenc）：在终端环境下使用 Vim 时，通过 termencoding 项来告诉 Vim 终端所使用的编码。

保存文件时使用 `:set fileencoding=utf-8` 来使用指定的编码进行保存。

# 自动配置选项

*:h auto-setting*

## modeline

有两种形式的 modeline：

- `[text]{white}{vi:|vim:|ex:}[white]{options}`
  - `text`：任意文本。
  - `white`：空格或 tab。
  - `options`：若干选项，用空格或“:”分隔，每个选项都被作为 `:set` 的命令。
- `[text]{white}{vi:|vim:|Vim:|ex:}[white]se[t] {options}:[text]`
  - `se[t]`：当使用“Vim”时必须用 `set`。

`ex:` 和 `Vim:` 前的空格是必需的，`vi:` 和 `vim:` 前的空格可选（为了兼容 3.0 版本）。

# 自定义命令

用户自定义命令必须首字母大写，来避免与内建命令冲突，除了 `:Next`、`:X`，它们不能用于自定义命令。`:Print` 也是内建命令，但因为已废弃所以可以覆盖。

调用命令时可以用缩写，若根据缩写找到的命令不唯一则会报错。内建命令总是优先。

- `:[verbose] com[mand] [{cmd}]` 列出所有用户定义的命令。展示结果前两列的符号：
  - `!` 该命令带有 `-bang` 属性。
  - `"` 该命令带有 `-register` 属性。
  - `b` 该命令仅在当前缓冲区有效。
  - `verbose` 额外显示命令的定义处。
  - `{cmd}` 仅展示以 `{cmd}` 开头的命令。
  - `:filter {subStr} command` 仅展示包含 `{subStr}` 的命令。
- `:com[mand][!] [{attr}...] {cmd} {rep}` 定义一个命令。名称为 `{cmd}`，命令内容为 `{rep}`（其中可能包含代替换的文本），属性为 `{attr}`。若命令已存在会报错。使用 `!` 强制覆盖已有命令。
  -
- `:delc[ommand] {cmd}` 删除命令。
- `comc[lear]` 删除所有用户定义命令。

## 替换文本

命令执行前会自动替换命令内容中的 `<...>` 占位符。若要避免被替换，需用 `<lt>` 代替 `<`。

- `<line1>` 命令处理范围的开始行。
- `<line2>` 命令处理范围的结束行。
- `<range>` 命令范围这个参数本身的参数个数：0，1，2。
- `<count>` 传入的计数值。
- `<bang>` 若命令执行时带有 `!` 则会展开为 `!`，否则不展开。
- `<mods>` 命令修饰符，若执行时没带则不展开。`:aboveleft`, `:belowright`, `:botright`, `:browse`, `:confirm`, `:hide`, `:keepalt`, `:keepjumps`, `:keepmarks`, `:keeppatterns`, `:leftabove`, `:lockmarks`, `:noswapfile`, `:rightbelow`, `:silent`, `:tab`, `:topleft`, `:verbose`, `:vertical`
- `<reg>` `<register>` 寄存器，若执行时没带则不展开。
- `<args>` 命令参数，但不包含计数和寄存器参数。
- `<lt>` 代表 `<`，用来保留 `<...>` 类文本避免被替换。
- `<q-{abc}>` 若某个转义序列以“q-”开始，则它整体会被引号包裹起来，作为一个字符串。
- `<f-args>` 将命令入参根据空格分割成列表然后作为函数入参。`:h <f-args>`

## 命令属性

### 参数处理

- 不显式声明参数。认为是不带参数，若调用时带参数会报错。
- `-nargs=0` 不带参数，同上。
- `-nargs=1` 有且仅有一个参数，包括空格。
- `-nargs=*` >= 0 个参数，空格分隔。
- `-nargs=?` 0 或 1 个参数。
- `-nargs=+` >= 1 个参数。

若仅有一个参数，则空格会作为参数的一部分；若有多个参数，空格和 tab 会作为参数间的分隔符。

参数会作为文本传入命令然后解析，而不是作为表达式先解析再传入命令。也就是说若将变量作为实参，那么会在命令定义的作用域内查找该变量，而不是在命令的调用处的作用域内。

### 补全行为

命令的参数默认不会被补全，除非声明下列一个或多个参数。

- `-complete=arglist` 参数列表中的文件名
- `-complete=augroup` 自动命名组
- `-complete=buffer` 缓冲区名称
- `-complete=behave` :behave 子选项
- `-complete=color` 颜色方案
- `-complete=command` Ex 命令及其参数
- `-complete=compiler` 编译器
- `-complete=cscope` :cscope 子选项
- `-complete=dir` 目录名
- `-complete=environment` 环境变量名
- `-complete=event` 自动命令事件
- `-complete=expression` Vim 表达式
- `-complete=file` 文件名和目录名
- `-complete=file_in_path` “path”中的文件名和目录名
- `-complete=filetype` 文件类型名
- `-complete=function` 函数名
- `-complete=help` 帮助主题
- `-complete=highlight` 高亮组
- `-complete=history` :history 子选项
- `-complete=locale` locale 名 (和 `locale -a` 给出的相同)
- `-complete=mapclear` 缓冲区参数
- `-complete=mapping` 映射名
- `-complete=menu` 菜单
- `-complete=messages` :messages 子选项
- `-complete=option` 选项
- `-complete=packadd` 可选包名
- `-complete=shellcmd` Shell 命令
- `-complete=sign` :sign 子选项
- `-complete=syntax` 语法文件名
- `-complete=syntime` :syntime 子选项
- `-complete=tag` 标签
- `-complete=tag_listfiles` 标签，但敲入 `<C-d>` 时显示文件名
- `-complete=user` 用户名
- `-complete=var` 用户变量
- `-complete=custom,{func}` 用户自定义补全，通过 `{func}` 定义。
- `-complete=customlist,{func}` 用户自定义补全，通过 `{func}` 定义。

### 范围处理

- `-range` 允许范围，默认当前行
- `-range=%` 允许范围，默认全文
- `-range={n}` 计数（默认 n），用法类似 `:3{cmd}`
- `-count=[{n}]` 计数（默认 n，若不声明则默认 0），用法类似 `:3{cmd}` 或 `:{cmd} 3`
- `-addr=lines` 将范围应用于行（默认）
- `-addr=arguments` 将范围应用于参数
- `-addr=buffers` 将范围应用于缓冲区，包括未加载的
- `-addr=loaded_buffers` 将范围应用于已加载的缓冲区
- `-addr=windows`	将范围应用于窗口
- `-addr=tabs` 将范围应用于标签页

### 特殊情况

- `-bang` 命令接受强制执行 `!`
- `-bar` 命令接受管道符 `|` 和后续命令。此时命令参数中不允许出现 `|`。同时会检查 `"` 作为注释的开始。
- `-register` 命令的第一个参数可以是一个可选的寄存器名（类似 `:del`）
- `-buffer` 命令仅在当前缓冲区可用
