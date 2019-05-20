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

# 减少媒体查询的建议

- 用百分比长度代替固定长度。若做不到，也应尝试用与窗口相关的单位(`vw`, `vh`, `vmin`, `vmax`)。
- 当需要在较大分辨率下得到固定宽度时，使用 `max-width` 而不是 `width`，因为它可以适应较小的分辨率而无需使用媒体查询。
- 不要忘记为替换元素(比如 `img`、`object`、`video`、`iframe` 等) 设置一个 `max-width`，值为 100%。
- 要将背景图铺满容器可用 `background-size: cover`。但是处于网络带宽的考虑，不应将大图缩小来显示。
- 当元素以行列式进行布局时，让窗口宽度来决定列的数量。比如使用 `Flexbox` 或 `display: inline-block` 加上常规的文本折行行为。
- 使用多列文本时，指定 `column-width` 而不是 `column-count` 可以在较小的屏幕上自动显示为单列布局。

# 合法颜色值

可选的指定方式：

- 十六进制：`#RRGGBB`
- RGB 颜色：`rgb(R, G, B)` RGB 分别为 0~255 的值或 0%~100% 的百分比值。
- RGBA 颜色：RGB 颜色在 alpha 通道的延伸（透明度）。`rgba(R, G, B, A)` A 是 0.0~1.0 之间的值。
- HSL 色彩：分别表示色调、饱和度、亮度。
  - 色调是在色轮上的位置。0 或 360 是红，120 是绿，240 是蓝。
  - 饱和度是 0%~100% 的百分比，分别是灰色阴影和全彩。
  - 亮度是一个 0%~100% 的百分比，分别是黑色和白色。
- HSLA 颜色：HSL 颜色在 alpha 通道的延伸。`hsla(H, S, L, A)` A 是 0.0~1.0 之间的值。
- 预定义/跨浏览器的颜色名称：147 种用名称预定义的颜色。

# 背景

- `background-color` 背景颜色
  - 十六进制 `#ff0000`
  - RGB `rgb(255,0,0)`
  - 名称 `red`
- `background-image` 背景图像，默认会平铺重复显示来覆盖整个元素实体。
- `background-repeat` 背景图平铺的重复方式。
- `background-position` 背景图位置。
- `background` 背景的简写属性。属性间空格分隔，顺序为：
  1. `background-color`
  2. `background-image`
  3. `background-repeat`
  4. `background-attachment`
  5. `background-position`

# 文本格式

- `color` 文本颜色
- `direction` 文本方向
- `letter-spacing` 字符间距
- `line-height` 行高
- `text-align` 对齐方式：center、right、left、justify(左右对齐)
- `text-decoration` 文本修饰，用来设置或删除文本的装饰。一般用来删除链接的下划线。
- `text-indent` 文本缩进
- `text-shadow` 文本阴影
- `text-transform` 文本转换。用于转换大小写。
- `unicode-bidi` 文本是否被重写
- `vertical-align` 元素的垂直对齐
- `white-space` 元素中空白的处理方式
- `word-spacing` 字间距

# 字体

有两种类型的字体系列名称：

- 通用字体系列：拥有相似外观的字体系统组合，如 Serif 和 Monospace。
- 特定字体系列：一个特定的字体系列，如 Times 或 Courier。

