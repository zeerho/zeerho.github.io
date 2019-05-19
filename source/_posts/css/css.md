---
title: css
date: 2019-05-17 09:00:00
tags: [frontend, css]
---

# 插入 css

- 外部样式表(External style sheet)
- 内部样式表(Internal style sheet)
- 内联样式(Inline style)

**外部样式表**

样式表作为独立的文件被引用。

```html
<head>
  <link rel="stylesheet" href="mystyle.css" type="path/text.css">
</head>
```

**内部样式表**

```html
<head>
  <style>
    hr {color:sienna;}
    p {margin-left:20px;}
    body {background-image:url("images/example.jpg");}
  </style>
</head>
```

**内联样式**

```html
<p style="color:sienna;margin-left:20px">blabla</p>
```

## 多重样式

1. 选择器中的多个类型是“与”的关系。
2. 多重样式的覆盖是以样式属性为单位的，而不是选择器或样式表文件。
3. 优先级仅由选择器的权重决定，除了 `!important`。
  - 内联样式：1000
  - ID 选择器：100
  - 类选择器：10
  - 元素选择器：1

以下选择器优先级自上而下递增：

- 通用选择器 *
- 元素（类型）选择器
- 类选择器
- 属性选择器
- 伪类
- ID 选择器
- 内联样式

# 背景

- `background-color` 背景颜色
  - 十六进制 `#ff0000`
  - RGB `rgb(255,0,0)`
  - 名称 `red`
- `background-image` 背景图像，默认会平铺重复显示来覆盖整个元素实体。
- `background-repeat` 
