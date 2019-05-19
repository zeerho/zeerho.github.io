---
title: Vue
date: 2017-01-01 09:00:00
tags: [Javascript]
---

# 入门

```html
<html>
<head>
  <meta charset="utf-8">
  <title>Hello World!</title>
</head>
<body>
  <script src="https://unpkg.com/vue/dist/vue.js"/>

  <script>console.log('hello!');</script>
</body>
</html>
```

# 计算属性

- 计算属性的值基于它的依赖进行缓存，只在必要时才重新执行函数。
- 当函数中用到的某个属性变化时，计算属性的值会自动更新。
- 计算属性可以跟普通属性一样使用。
- 计算属性只有真正用于应用中时才会计算。
