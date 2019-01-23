---
title: vimscript
date: 2019-01-11 14:00:00
tags: [工具, vim]
---

# 变量

`let/unlet` 赋值/删除

<!-- more -->

## 作为变量的选项

`&{opt}` 其中 `&` 表示引用的是一个选项而不是一个同名的变量。

使用 `set` 来设置选项只能设置常量值，而 `let` 可以当作变量来处理。

```
let &textwidth=&textwidth+10
```

## 作为变量的寄存器

`@{reg}` 其中 `@` 表示引用的是一个寄存器而不是一个同名的变量。

## vim 预置的变量

`:h vim-variable`

# 变量作用域

`:h internal-variables`

`b:` 缓冲区
`w:` 窗口
`t:` 标签页
`g:` 全局
`l:` 函数
`s:` 脚本文件
`a:` 函数参数
`v:` 全局的 vim 预置变量

作用域命名空间本身可以当作字典来用：

```vimscript
for k in keys(s:)
  unlet s:[k]
endfor
```

# 函数

## function!

如果存在同名函数的话就覆盖（否则会报错）。

# 列表

```vimscript
" 异质元素集合
['foo', [3, 'bar']]

" 索引
echo ['a', 'b', 'c'][0]  " a
echo ['a', 'b', 'c'][-2] " b
" 字符串不能用负数索引但能用负数切割

" 切割
echo ['a', 'b', 'c', 'd'][0:2]   " a b c
echo ['a', 'b', 'c', 'd'][:2]    " a b c
echo ['a', 'b', 'c', 'd'][0:]    " a b c d
echo ['a', 'b', 'c', 'd'][:]     " a b c d
echo ['a', 'b', 'c', 'd'][0:9]   " 越界安全
echo ['a', 'b', 'c', 'd'][-2:-1] " c d

" 连接
['a'] + ['b'] " ['a', 'b']

" 内置函数
let arr = ['a']
call add(arr, 'b')              " ['a', 'b']
call len(arr)                   " 2
echo get(arr, 1, 'default val') " b
echo index(arr, 'b')            " 1，若不存在返回 -1 
echo join(arr)      " a b
echo join(arr, '-') " a-b
echo join(arr, '')  " ab
call reverse(arr)   " 转置
```

