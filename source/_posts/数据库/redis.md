---
title: redis
date: 2019-01-02 09:00:00
tags: [数据库]
---

# 安装

1. 解压后 `make`。
2. `make` 后在 `$REDIS_HOME/src` 目录下会有服务端 redis-server 和客户端 redis-cli。
3. 启动服务。`./redis-server [pathToConfig]` 可指定配置文件，缺省用默认配置 `$REDIS_HOME/redis.conf`。
4. 启动客户端。`./redis-cli` 然后开始与服务端交互。

# 数据结构

- `STRING` 可以是字符串、整数或浮点数。
  - 对整个字符串或其中一部分进行操作。
  - 对整数和浮点数进行自增或自减操作。
- `LIST` 一个链表，链表上每个节点包含一个字符串。
  - 从链表两端推入或弹出元素。
  - 根据偏移量对链表进行修剪（trim）。
  - 读取单个或多个元素；根据值查找或移除元素。
- `SET` 包含字符串的无序收集器，并且其中的字符串各不相同。
  - 添加、获取、移除单个元素。
  - 检查一个元素是否存在。
  - 计算交集、并集、差集。
  - 从集合中随机获取元素。
- `HASH` 包含键值对的无序散列表。值可以是字符串或数字。
  - 添加、获取、移除单个键值对。
  - 获取所有键值对。
- `ZSET` 字符串成员与浮点数分值之间的有序映射，元素的排列顺序由分值的大小决定。
  - 添加、获取、删除单个元素。
  - 根据分值范围或成员来获取元素。


# 命令

见 [官方文档](http://redis.io/commands)。可直接后接命令名来查询。

- `redis-cli [-p {port}] shutdown` 关闭服务端。
- `redis-cli -h {ip} -p {port} -a "mypass"` 登录远程服务端。
- `flushall` 删除所有库的所有数据。
- `type {key}` 判断值的类型。

## STRING 字符串

字符串可以存储的值的类型：

- 字节串
- 整数
- 浮点数

有需要的时候 Redis 会将整数转换成浮点数。整数的取值范围和系统的长整数相同（32 位/64 位系统分别是 32 位/64 位 有符号整数）。浮点数的取值范围与 IEEE 754 标准的双精度浮点数相同。

- `INCR {key}` 加 1。返回加后的字符串。
- `DECR {key}` 减 1。返回减后的字符串。
- `INCRBY {key} {int}` 加上指定整数。返回加后的字符串。
- `DECRBY {key} {int}` 减去指定整数。返回减后的字符串。
- `INCRBY {key} {float}` 加上指定浮点数，返回加后的字符串。
- `APPEND {key} {val}` 在当前值的末尾追加新值。
- `GETRANGE {key} {start} {stop}` 返回由范围内的字符组成的子串，包括范围首尾。
- `SETRANGE {key} {offset} {val}` 将 `{offset}` 开始的子串设定值。
- `GETBIT {key} {offset}` 将字节串看成是二进制位串，返回 `{offset}` 处的比特值。
- `SETBIT {key} {offset} {val}` 将字节串看成是二进制位串，设置 `{offset}` 处的比特值。
- `BITCOUNT {key} [{start} {stop}]` 统计二进制位串中 1 的数量。可以框定范围。
- `BITOP {operation} {destKey} {key} [{key}...]` 对若干二进制串执行按位操作（并、或、异或、非），将结果存入 `{destKey}`。


## LIST 链表

- `LPUSH/RPUSH {key} {val}` 推入链表左/右端。返回链表当前长度（包含此次推入的元素）。
- `LRANGE {key} {start} {stop}` 返回指定索引范围内的元素。若 `{start}` 大于链表末尾，则返回空表；若 `{stop}` 大于链表末尾，则把它当作链表末尾的索引来处理。负数表示从末尾开始反向索引。
- `LINDEX {key} {idx}` 返回指定索引的元素。负数表示反向索引。若索引越界，则返回 nil。
- `LPOP/RPOP {key}` 弹出左/右端的一个元素。当 `{key}` 不存在时返回 nil。

## SET 集合

- `SADD {key} {member} [{member}...]` 加入元素。返回本次操作真正新增的元素个数。
- `SMEMBERS {key}` 返回集合中所有元素。
- `SISMEMBER {key} {member}` 集合中是否包含指定元素。若包含返回 1；若不包含或 `{key}` 不存在返回 0。
- `SREM {key} {member} [{member}...]` 删除指定元素。返回本次操作真正删除的元素个数。

## HASH 散列

- `HSET {key} {field} {value}` 加入键值对。若 `{field}` 是新的则返回 1；若 `{field}` 已存在则返回 0。
- `HGET {key} {field}` 返回指定键的值。若 `{key}` 或 `{field}` 不存在则返回 nil。
- `HGETALL {key}` 返回散列中所有键值对。若 `{key}` 不存在则返回空表。
- `HDEL {key} {field} [{field}...]` 删除指定键的键值对。返回实际被删除的键值对数量。
- `HMGET {key} {field} [{field}...]` 获取若干键的值。
- `HMSET {key} {field} {val} [{field} {val}...]` 加入若干键值对。
- `HLEN {key}` 返回键值对数量。
- `HEXISTS {key} {field}` 给定键是否存在。
- `HKEYS {key}` 获取散列中的所有键。
- `HVALS {key}` 获取散列中的所有值。
- `HGETALL {key}` 获取散列包含的所有键值对。
- `HINCRBY {key} {field} {increment}` 将指定键的值加上某整数。
- `HINCRBYFLOAT {key} {field} {increment}` 将指定键的值加上某浮点数。


## ZSET 有序集合

- `ZADD {key} [NX|XX] [CH] [INCR] {score} {member} [{score} {member} ...]` 添加带有指定分值的成员。
  - `NX` 只新增，不更新已有的元素。
  - `XX` 不新增，只更新已有的元素。
  - `CH` 更改返回值的含义。本来是返回实际新增元素的数量，现在改为被修改元素的数量（包括新增的和分值产生变化的）。
  - `INCR` 此时 `zadd` 相当于 `zincby`，而且只能指定一组 `{score}` `{member}`。
- `ZRANGE {key} {start} {stop} [WITHSCORES]` 返回索引范围内的元素。按照分值由低到高排列，相同分值的按照字母顺序排列。
  - `WITHSCORES` 连带返回对应的分值。
- `ZREVRANGE {key} {start} {stop} [WITHSCORES]` 同 `ZRANGE`，只不过是逆序排列。
- `ZCARD {key}` 返回指定集合的基数。

# 集群

##
