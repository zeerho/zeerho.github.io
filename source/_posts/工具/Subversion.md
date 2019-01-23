---
title: Subversion
date: 2017-01-01 09:00:00
tags: [工具]
---

# 常用

`svn co --depth=empty "{address}"`

update depth:

- working copy

# 选项

## 全局选项

`--auto-props`: 覆盖 config 文件中的 `enable-auto-props` 选项。
`--change|-c {rev}`: 引用特定修订版本发生的修改。实际上是 `-r {rev-1}:{rev}` 的语法糖。
`--config-dir {dir}`: 指定配置目录（默认为 ~/AppData/Roaming/Subversion 或 ~/.Subversion）。
`--diff-cmd {cmd}`: 指定对比工具。
`--diff3-cmd {cmd}`: 指定合并工具。
`--dry-run`: 预览命令效果。
`--editor-cmd {cmd}`: 指定编辑器。
`--encoding {enc}`: 提交日志时指定编码。
`--extensions|-x {args}`: 指定传递给对比工具的参数，`{args}` 为多个参数时须用双引号包裹。只跟 `--diff-cmd` 配套使用。

# Subversion组件

- svn：命令行客户端程序。
- svnversion：用来显示工作副本的状态（即当前项目的修订版本）。
- svnlook：直接查看Subversion版本库的工具。
- svnadmin：建立、调整和修复Subversion版本库的工具。
- mod_dav_svn：Apache Http服务器的一个插件，使版本库可以通过网络访问。
- svnserve：一个单独运行的服务器程序，可以作为守护进程或由SSH调用。这是另一种使版本库可以通过网络访问的方式。
- svndumpfilter：过滤Subversion版本库转储数据流的工具。
- svnsync：一个通过网络增量镜像版本库的程序。

# 版本库的地址

**表1 不同的URL模式对应的访问方法**

|模式      |访问方法                                    |
|:---------|:-------------------------------------------|
|file:///  |直接版本库访问（本地磁盘）                  |
|http://   |通过配置Subversion的Apache服务器的WebDAV协议|
|https://  |与http://相似，但是包括SSL加密              |
|svn://    |通过svnserve服务自定义的协议                |
|svn+ssh://|与svn://相似，但通过SSH封装                 |
