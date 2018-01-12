---
title: Hexo
date: 2018-01-01 22:00:00
tags: [工具]
---

本文根据网上各处的教程和自身的实践，记录了用 Hexo 搭建博客的各方面信息。

# 搭建过程概述

## 环境准备

1. Git
2. Node.js
3. Hexo
4. Hexo 主题

<!-- more -->

### 准备 Git

1. 下载并安装 https://git-scm.com/downloads
2. 设置用户名和邮箱
```
git config --global user.name "MyName"
git config --global user.email "MyEmail@gmail.com"
```
3. 生成密钥，输入下述命令然后一路回车
```
ssh-keygen -t rsa -C "MyEmail@gmail.com"
```
4. 在 Github 上注册，设置公钥，新建 repository，名为 `MyName.github.io`


### 准备 Node.js

1. 下载并安装 http://nodejs.cn/
2. 在命令行中依次执行 `node -v` 和 `npm -v` 来查看是否成功安装

### 准备 Hexo

1. `
1. 进入用来放博客的文件夹，比如 `C:\workspace\blog` （下面涉及到这个文件夹的地方都要替换为你自己的文件夹）
2. 在命令行中执行 `npm install -g hexo-cli`、`hexo init`、`npm install`、`hexo g`、`hexo s`
3. 访问 `localhost:4000`，若以上步骤没问题则能看到 Hexo 的默认页面
4. 回命令行中按 `Ctrl-C` 来关闭刚才 `hexo s` 启动的本地服务
5. 配置部署方式。打开刚才的 `C:\workspace\blog`，编辑 _config.yml
```
deploy:
  type: git
  repo: git@github.com:MyName/MyName.github.io.git
  branch: master
```
6. 为了使用 `hexo d` 来部署到 Github 上，需要安装
`npm install hexo-deployer-git --save`

### Hexo 主题

主要有两种方式下载主题文件，一是下载别人的压缩包，二是把 Github 上的 theme 项目 clone 下来。
两种方式的本质是一样的。推荐第二种，这样既可以把 theme 项目 fork 过来方便自己学习、修改，也可以利用 Git 来管理。

1. 把下载的 theme 文件夹重命名。比如我用的是 NexT 主题，就重命名为 next。
2. 把 next 文件夹放到 `C:\workspace\blog` 下面。
3. 修改站点配置文件
  1. 打开 `C:\workspace\blog\_config.yml`
  2. 找到 `theme` 字段，改为 `theme: next`，这里的 `next` 对应上面重命名的文件夹名字

---
## 配置

### 标签 tags

1. 在 `C:\workspace\blog\source` 下面新建文件夹 `tags`
2. 在 `tags` 中新建文件 `index.md`，内容为
```
title: Tags
date: 2018-01-01 15:00:00
type: "tags"
comments: false
---
```

---
## 发布博文

### 新建
`hexo new [layout] xxx`
hexo 会在 _posts 目录下自动新建一个 xxx.md 的文件。若指定了布局模板 layout，则 xxx.md 中会预先填充了该模板的内容。
scaffolds 目录下有默认的布局模板。也可以仿照着创建自定义的模板。

或者也可以将已有的 xxx.md 放到 `C:\workspace\blog\source\_posts` 下面。
也可以在这个目录下面自定义一些文件夹来分门别类地存放。

