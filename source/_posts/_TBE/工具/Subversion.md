---
title: Subversion
date: 2017-01-01 09:00:00
tags: [工具]
---

# 常用

`svn co --depth=empty "{address}"`

update depth:

- working copy

---
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
|:---------|:------------------------------------------:|
|file:///  |直接版本库访问（本地磁盘）                  |
|http://   |通过配置Subversion的Apache服务器的WebDAV协议|
|https://  |与http://相似，但是包括SSL加密              |
|svn://    |通过svnserve服务自定义的协议                |
|svn+ssh://|与svn://相似，但通过SSH封装                 |
