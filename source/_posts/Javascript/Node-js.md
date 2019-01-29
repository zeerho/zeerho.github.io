---
title: Node-js
date: 2018-12-14 14:02:00
tags: [JavaScript, node]
---

# 模块化

## 使用简介

```js
// 文件名 hello-there.js
// 在模块中暴露方法、变量
exports.hello = function () {
  console.log("hello!");
}

// 在另一个文件中引入模块
// 这里拿到的就是指定模块的 `exports` 对象
var helloThere = require("./maybe-paths-here/hello-there");
helloThere.hello();
```

## npm

### 使用

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
- `npm search {keyWord}`: 搜索模块名字和描述。

### package.json

```json

dadada
```

### 配置

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

## 使用模块

### 查找模块

1. 内置模块。
2. 若模块名以路径开始（`./`、`../`、`/`），则在指定路径搜索。
  1. 若指定了 `.js` 扩展名，则直接加载该文件。
  2. 否则查找基于同名文件夹的模块。
  3. 若没有，则添加扩展名 `.js`、`.json`、`.node`，依次尝试加载。（`.node` 会被编译成附加模块）
3. 若模块名不是以路径开始，则在 `./node_modules` 下查找。
4. 若没有，则以 Node 自己的方式在当前位置的目录树下搜索 `node_modules` 文件夹。
5. 若没有，则在一些默认路径下搜索，如 `/usr/lib`、`/usr/local/lib`。
6. 若没有，抛出错误。

### 循环引用

发生循环引用时，Node 先简单地返回一个尚未完成初始化的对象到引用处。在所有引用处都拿到对象之后再依次进行对象初始化。

## 编写模块

1. 创建一个文件夹来存放模块内容。
2. 添加 `package.json` 文件。其中至少包含当前模块名称和一个主要 js 文件（作为加载入口）。
3. 若未找到 `package.json` 或没指定主要 js 文件，则会查找 `index.js` 或编译后的附加模块 `index.node`。
