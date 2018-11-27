---
title: oracle
date: 2018-11-15 09:00:00
tags: [数据库, oracle]
---

# sqlplus 登录的 4 种方式

1. sqlplus / as sysdba
    操作系统认证，不需要数据库服务器启动listener，也不需要数据库服务器处于可用状态。比如我们想要启动数据库就可以用这种方式进入
    sqlplus，然后通过startup命令来启动。
2. sqlplus username/password
    连接本机数据库，不需要数据库服务器的listener进程，但是由于需要用户名密码的认证，因此需要数据库服务器处于可用状态才行。
3. sqlplus usernaem/password@orcl
    通过网络连接，这是需要数据库服务器的listener处于监听状态。此时建立一个连接的大致步骤如下　
　　a. 查询sqlnet.ora，看看名称的解析方式，默认是TNSNAME　　
　　b. 查询tnsnames.ora文件，从里边找orcl的记录，并且找到数据库服务器的主机名或者IP，端口和service_name　　
　　c. 如果服务器listener进程没有问题的话，建立与listener进程的连接。　　
　　d. 根据不同的服务器模式如专用服务器模式或者共享服务器模式，listener采取接下去的动作。默认是专用服务器模式，没有问题的话客户端
            就连接上了数据库的server process。
　　e. 这时连接已经建立，可以操作数据库了。
4.sqlplus username/password@//host:port/sid
　　用sqlplus远程连接oracle命令(例：sqlplus risenet/1@//192.168.130.99:1521/risenet)

# 查看版本信息

```sql
select * from v$version;
```
