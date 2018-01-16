---
title: Eclipse
date: 2017-01-01 09:00:00
tags: [工具]
---

# 在标题栏显示当前工作空间路径

在 eclipse 的 `快捷方式->属性->快捷方式->目标` 中，在末尾加上 ` -showlocation`，注意开头有一个空格。若 eclipse 的父路径中包含空格，则目标栏内的路径会被包含在双引号中，此时需要把 ` -showlocation` 加在闭引号后面。
或在 eclipse.ini 中，在“-vmargs”前某一行加入 `-showlocation`。

---
# sublipse 选项显示不全

问题：`右键项目->Team->` 中只有 `Apply Patch` 和 `Share Project` 选项。
原因：项目未连接到版本控制工具。
解决：

- 点击 `Share Project`，然后根据提示操作，与版本控制工具连接。
**或**
- 将 workspace 中的项目删除后（源代码不删），重新导入。这样 eclipse 会自动将之前与版本控制工具连接过的项目重新连接。

---
# eclipse 版本与 JDK 版本对应关系

|eclipse 版本|JDK 版本|
|:-----------|:-------|
|4.6 Neon    |8       |
|4.5 Mars    |7       |
|4.4 Luna    |7       |
|4.3 Kepler  |6       |
