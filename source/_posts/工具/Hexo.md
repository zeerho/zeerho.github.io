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

[Hexo 中文文档](https://hexo.io/zh-cn/docs/)

1. 进入用来放博客的文件夹，比如 `C:\workspace\blog` （下面涉及到这个文件夹的地方都要替换为你自己的文件夹）
2. 全局安装 hexo-cli `npm install -g hexo-cli`。
3. `hexo init` 初始化全部文件（迁移博客时不要执行，会覆盖掉原先的配置等文件）、`npm install`
4. `hexo g` 生成静态文件
5. `hexo s` 部署并启动服务
6. 访问 `localhost:4000`，若以上步骤没问题则能看到 Hexo 的默认页面
7. 回命令行中按 `Ctrl-C` 来关闭刚才 `hexo s` 启动的本地服务
8. 配置部署方式。打开刚才的 `C:\workspace\blog`，编辑 `_config.yml`
```
deploy:
  type: git
  repo: git@github.com:MyName/MyName.github.io.git
  branch: master
```
9. 为了使用 `hexo d` 来部署到 Github 上，需要安装 `npm install hexo-deployer-git --save`

### Hexo 主题

主要有两种方式下载主题文件，一是下载别人的压缩包，二是把 Github 上的 theme 项目 clone 下来。
两种方式的本质是一样的。推荐第二种，这样既可以把 theme 项目 fork 过来方便自己学习、修改，也可以利用 Git 来管理。

1. 把下载的 theme 文件夹重命名。比如我用的是 NexT 主题，就重命名为 next。
2. 把 next 文件夹放到 `C:\workspace\blog` 下面。
3. 修改站点配置文件
  1. 打开 `C:\workspace\blog\_config.yml`
  2. 找到 `theme` 字段，改为 `theme: next`，这里的 `next` 对应上面重命名的文件夹名字

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

### 插入图片

最简单的是使用绝对路径。在 source 下创建 images 文件夹。

`![](/images/abc.jpg)`

可以使用子目录来保持图片跟文档的目录结构相同，方便归类和查找。但这样的话若改动目录结构就需要同时更改文档和图片的目录结构，还需要修改文档中引用图片的路径。

更灵活的方式是使用相对路径的方式引用图片。

1. 将 `_config.yml` 中的 post_asset_foler 改为 true。
2. 在文档的同级目录下创建同名目录，里面存放图片。
3. 引用图片 `![](abc.jpg)`

## 发布博文

### 新建

`hexo new [layout] xxx`

hexo 会在 `_posts` 目录下自动新建一个 xxx.md 的文件。若指定了布局模板 layout，则 xxx.md 中会预先填充了该模板的内容。

scaffolds 目录下有默认的布局模板。也可以仿照着创建自定义的模板。

或者也可以将已有的 xxx.md 放到 `C:\workspace\blog\source\_posts` 下面。 也可以在这个目录下面自定义一些文件夹来分门别类地存放。

# 问题

遇到 3 个连续的 `{` 时，hexo 解析会报错，详细参考 [这篇文章](!http://mlnote.com/2016/09/03/solved-mathjex-problem/)

`.deploy_git/` 下在初始化的时候会生成 `.git/`，后续发布的就是这里的内容。若没有这个 `.git/` 则发布的是外面的源文件。若手动删除了这个 `.git/`，可以手动 `git init` 一下。
