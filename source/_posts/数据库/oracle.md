---
title: oracle
date: 2018-11-15 09:00:00
tags: [数据库, oracle]
---

# TNS 介绍

Transparence Network Substrate 透明网络底层。

TNS 是用来管理 oracle 服务端和客户端连接的一个工具。大部分情况下客户端连接服务端都要配置 TNS（少部分情况比如通过 JDBC 连接）。要通过 TNS 连接服务端，客户端必须安装 Oracle Client。

TNS 配置文件分为服务端和客户端：

- 服务端：listener.ora、sqlnet.ora、tnsnames.ora 等。
- 客户端：sqlnet.ora、tnsnames.ora。

默认在 `$ORACLE_HOME/network/admin/` 下。

- listener.ora: 配置服务端的监听器，用来监听客户端的连接请求。默认是 1521 端口。
- sqlnet.ora: 管理和约束 TNS 连接的配置。
- tnsnames.ora: 配置连接。

## listener.ora

```
LISTENER = 
  (DESCRIPTION_LIST =
    (DESCRIPTION = 
      (ADDRESS = (PROTOCOL = TCP)(HOST = {host})(PORT = 1521))
      (ADDRESS = (PROTOCOL = IPC)(KEY = EXTPROC1521))
    )
  )
```

这里没列出 `SID_LIST_LISTENER` 部分，因为从 9i 开始引入了动态监听服务注册，数据库启动时会自动注册实例到监听列表。

## tnsnames.ora

```
{tnsName} =
  (DESCRIPTION =
    (ADDRESS_LIST =
      (ADDRESS = (PROTOCOL = {protocol})(HOST = {host})(PORT = {port}))
    )
    (CONNECT_DATA =
      (SERVICE_NAME = {serviceName})
    )
  )
```

- {tnsName}: TNS 名。
- `DESCRIPTION`: 服务端的监听地址信息。
  - {protocol}: 一般是 tcp。
  - {host}: 服务端 ip。在局域网内也可以用计算机名。
  - {port}: 默认 1521。
- `CONNECT_DATA`: 要连接的数据以及连接方式（专用或共享）。


# sqlplus 登录的 4 种方式

1. `sqlplus / as sysdba`
操作系统认证，不需要数据库服务器启动 listener，也不需要数据库服务器处于可用状态。比如想要启动数据库就可以用这种方式进入 sqlplus，然后通过 `startup` 命令来启动。
2. `sqlplus {username}/{password}`
连接本机数据库，不需要数据库服务器的 listener 进程，但是由于需要用户名密码的认证，因此需要数据库服务器处于可用状态。
3. `sqlplus {username}/{password}@{host}/{instanceName} [as sysdba]`
通过网络连接，这时需要数据库服务器的 listener 处于监听状态。`as sysdba` 表示作为 dba 登录。也可以用 tnsname 代替 `{host}/{instanceName}`。此时建立一个连接的大致步骤如下：
  1. 查询 sqlnet.ora，看看名称的解析方式，默认是 TNSNAME。
  2. 查询 tnsnames.ora 文件，找到具体的连接方式。
  3. 如果服务器 listener 进程没有问题的话，建立与 listener 进程的连接。
  4. 根据不同的服务器模式如专用服务器模式或者共享服务器模式，listener 采取接下去的动作。默认是专用服务器模式，没有问题的话客户端就连接上了数据库的服务端进程。
4. `sqlplus /nolog` `conn {username}/{password}[@{host}/{instanceName}` 依次执行两条命令，这样创建的进程是不带用户名密码的。

# 客户端安装配置

1. 添加环境变量 `TNS_ADMIN`，值为 tnsnames.ora 文件所在路径。如果安装了 ORACLE 并且设置了 `ORACLE_HOME` 环境变量，那么会自动在 `%ORACLE_HOME%/network/admin/` 下查找 tnsnames.ora 文件。
2. 添加环境变量 `NLS_LANG = SIMPLIFIED CHINESE_CHINA.ZHS16GBK` （`AMERICAN_AMERICA.US7ASCII` 是 ASCII 编码类型）
3. 配置 plsql: tools - preferences - connection 
  - Oracle Home=$oracleclient
  - OCI library=$oracleclient\oci.dll  

# 查看版本信息

```sql
select * from v$version;
```
