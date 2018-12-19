---
title: Node-js
date: 2018-12-14 14:02:00
tags: [JavaScript, node]
---

# npm

- `npm -v`
- `npm init [--yes]`：初始化一个 `package.json`；`--yes` 使用默认配置。
  - `name`：模块名，要求：
    - 全部小写
    - 没有空格
    - 允许破折号和下划线
  - `description`：若为空会从当前目录下的 `README.MD` 或 `README` 读取第一行作为内容
  - `scripts`：常用命令
- `npm install {package_name} [--save|save-dev]`：安装指定包到当前目录下的 `node_modules` 目录。`--save|save-dev` 安装包的同时更新 `package.json`，分别对应生产环境和开发环境。
  - `npm install npm@latest -g`：单独更新 npm。`-g` 表示全局模块。
- `npm uninstall {package_name} [--save]`：卸载模块。`--save` 卸载的同时也从 `package.json` 中删除。
- `npm ls [--depth=0]` 查看依赖情况。
- `npm outdated [-g] [--depth=0]`：查看过期包。
- `npm update [-g] [{package_name}]`：更新模块。

## 配置

- `npm config set {key} {val} [-g]` 或 `npm set {key} [-g]`：用户级别配置。`-g` 系统级别（$prefix/npmrc）。
- `npm config get {key}` 或 `npm get {key}`
- `npm config delete {key}`
- `npm config list [-l] [--json]`

**配置加载顺序**

1. 命令行参数
2. 环境变量
3. npmrc
  1. 项目级别：/myProj/.npmrc
  2. 用户级别：~/.npmrc
  3. 全局级别：$PREFIX/etc/npmrc
  4. npm 内置：/pathToNpm/npmrc

**配置源**

- `npm install {pkg} --registry {url}`：安装包的时候临时使用。
- `npm config set registry {url}`：用户级别。
  - `npm config get registry` 或 `npm info {pkg}`：验证是否配置成功。
- 使用 cnpm：
  - `npm install -g cnpm --registry={url}`：安装 cnpm。
  - `cnpm install {pkg}`：使用 cnpm 安装包。

