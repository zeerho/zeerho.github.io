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

<!-- more -->

**集群**

1. 查看是否安装 Ruby：`ruby -v`
2. 安装 Ruby：`apt-get install ruby-full`

# 数据结构

键总是字符串类型，值可以是以下五种类型之一。

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
- `ZSET` 字符串成员与浮点数分值（IEEE 754 双精度浮点数）之间的有序映射，元素的排列顺序由分值的大小决定。
  - 添加、获取、删除单个元素。
  - 根据分值范围或成员来获取元素。

## 字符串

redis 使用自己的字符串格式：简单动态字符串（simple dynamic string, SDS）。

```c
struct sdshdr {
  // buf 中已使用的字节数
  int len;
  // buf 中未使用的字节数
  int free;
  // 字节数组，用于保存字符串
  char buf[];
};
```

`buf` 保存的字节数组遵循 c 字符串的惯例，以空字节 '\0' 结尾，为的是方便重用 c 字符串函数库。该空字节不算在 `len` 里。

### SDS 的优点

**常数复杂度获取字符串长度**

直接访问 `len` 字段获取长度。而 C 字符串需要遍历整个字符串。

**防止缓冲区溢出**

C 字符串假设用户在拼接字符串前已经检查缓冲区是否会溢出。SDS API 会自动检查和扩展缓冲区尺寸。

**减少修改字符串时的内存重分配**

- 空间预分配：SDS 进行增长操作时，会分配额外的空间。
  - 若 SDS 修改后 `len` 小于 1MB，那么 `free` 将分配为等于修改后的 `len`。
  - 若 SDS 修改后 `len` 大于等于 1MB，那么 `free` 将分配到 1MB。
- 惰性空间释放：SDS 被截短后不会立即释放无用空间，而是由 `free` 字段记录，供后续 SDS 拼接时使用。SDS 也提供了 API 进行真正的空间释放。

**二进制安全**

C 字符串中的字符必须符合某种编码，并且除了结尾外不能存空字节，因此不能存文本之外的数据。SDS 以二进制形式直接存取数据，它以 `len` 字段判断字符串结尾，不对字符串中的内容做任何的假设和处理。

### SDS 主要 API

|函数       |作用                                              |时间复杂度                         |
|:----------|:-------------------------------------------------|:----------------------------------|
|sdsnew     |创建包含指定 C 字符串的 SDS                       |O(N), N 为 C 字符串长度            |
|sdsempty   |创建一个空的 SDS                                  |O(1)                               |
|sdsfree    |释放 SDS                                          |O(N), N 为 SDS 长度                |
|sdslen     |已用字节数                                        |O(1)                               |
|sdsavail   |未用字节数                                        |O(1)                               |
|sdsdup     |创建一个 SDS 副本                                 |O(N), N 为 SDS 长度                |
|sdsclear   |清空 SDS 内的字符串内容                           |O(1)                               |
|sdscat     |将指定 C 字符串拼接到 SDS 末尾                    |O(N), N 为被拼接的 C 字符串长度    |
|sdscatsds  |将指定 SDS 字符串拼接到 SDS 末尾                  |O(N), N 为被拼接的 SDS 字符串长度  |
|sdscpy     |用指定 C 字符串覆盖 SDS 内容                      |O(N), N 为被复制 C 字符串长度      |
|sdsgrowzero|用空字符将 SDS 扩展至指定长度                     |O(N), N 为新增的字节数             |
|sdsrange   |保留 SDS 给定区间内的数据，其他数据会被覆盖或清除 |O(N), N 为保留的字节数             |
|sdstrim    |从指定 SDS 中清除所有在指定 C 字符串中出现过的字符|O(N^2), N 为指定 C 字符串长度      |
|sdscmp     |对比两个 SDS 是否相同                             |O(N), N 为两个 SDS 中较短那个的长度|


## 链表

```c
// adlist.h/listNode
typedef struct listNode {
  // 前置节点
  struct listNode *prev;
  // 后置节点
  struct listNode *next;
  // 节点的值
  void *value;
} listNode;
```

redis 链表特性：

- 双端：是双向链表。
- 无环
- 带表头指针、表尾指针、长度计数器：`list` 结构带有表头指针、表尾指针、链表长度计数器。
- 多态：节点值的类型是 `void*`，可以存放各种类型数据。

## 字典

以哈希表作为实现

```c
// 哈希表
// dict.h/dictht
typedef struct dictht {
  // 哈希表数组
  dictEntry **table;
  // 哈希表大小
  unsigned long size;
  // 哈希表大小掩码，用于计算索引值
  // 总是等于 size - 1
  unsigned long sizemark;
  // 已有节点数量
  unsigned long used;
} dictht;

// 节点
// dict.h/dictEntry
typedef struct dictEntry {
  // 键
  void *key;
  // 值
  union {
    void *val;
    unint64_t u64;
    int64_t s64;
  } v;
  // 指向下个节点
  struct dictEntry *next;
} dictEntry;

// 字典
// dict.h/dict
typedef struct dict {
  // 类型特定函数
  dictType *type;
  // 私有数据
  void *privdata;
  // 哈希表
  dictht ht[2]; // ht[0] 常用，ht[1] 仅用在 rehash 时
  // rehash 索引，不在 rehash 时值为 -1
  long rehashidx;
} dict;
```

- 使用 MurmurHash2 算法来计算哈希值。有点是随机分布性好，速度快。
- 使用链地址法（外部拉链法）解决哈希碰撞问题。

### rehash

1. 为 `ht[1]` 分配空间：
  - 若是扩展操作，则 `ht[1]` 的大小为第一个大于等于 `ht[0].used * 2` 的 2^n。
  - 若是收缩操作，则 `ht[1]` 的大小为第一个大于等于 `ht[0].used` 的 2^n。
2. 将 `ht[0]` 中所有键值对 rehash 到 `ht[1]` 中。
3. 释放 `ht[0]`，将 `ht[1]` 置为 `ht[0]`，并在 `ht[1]` 新建一张空表。

**哈希表的扩展与收缩**

> 负载因子 = 已存节点数量 / 哈希表大小

满足以下任一条件时会开始哈希表扩展操作：

- 当前未在执行 `BGSAVE` 和 `BGREWRITEAOF`，且负载因子 >= 1。
- 当前正在执行 `BGSAVE` 和 `BGREWRITEAOF`，且负载因子 >= 5。

以上两个命令会 fork 子进程，而 linux 采用写时复制来优化子进程的效率。提高负载因子会降低子进程产生写的概率，从而优化性能。

满足以下条件时开始哈希表收缩操作：

- 负载因子 <= 0.1。

**渐进式 rehash**

1. 为 `ht[1]` 分配空间，让字典同时持有两个哈希表。
2. 将字典中的索引计数器变量 `rehashidx` 置为 0，表示 rehash 开始。
3. 在 rehash 期间，每次对字典执行增删改查时，除了执行该操作外（删该查在两张表操作，增在 `ht[1]`），还会将 `ht[0]` 在 `rehashidx` 位置的所有键值对 rehash 到 `ht[1]` 上，然后 `rehashidx` 加 1。
4. 最终所有的键值对都完成了 rehash，此时将 `rehashidx` 置为 -1，表示 rehash 结束。

## 跳跃表

`redis.h/zskiplistNode`、`redis.h/zskiplist`

# 命令

见 [官方文档](http://redis.io/commands)。可直接后接命令名来查询。

## 数据库

- `redis-cli [-p {port}] shutdown` 关闭服务端。
- `redis-cli -h {ip} -p {port} -a "mypass"` 登录远程服务端。
- `flushall` 删除所有库的所有数据。
- `type {key}` 判断值的类型。
- `del {key} [{key}...]` 不存在的会被忽略。返回删除的数量。

## STRING 字符串

字符串可以存储的值的类型：

- 字节串
- 整数
- 浮点数

有需要的时候 Redis 会将整数转换成浮点数。整数的取值范围和系统的长整数相同（32 位/64 位系统分别是 32 位/64 位 有符号整数）。浮点数的取值范围与 IEEE 754 标准的双精度浮点数相同。

- `GET {key}` 获取字符串值。若值不是字符串则返回一个错误。
- `SET {key} {val} [expiration {EX} seconds|{PX} milliseconds] [NX|XX]` 设置字符串。会覆盖原有值，且无视原有值的类型。
  - 2.6.12 版本开始支持以下选项：
  - `EX` 过期时间（秒）。
  - `PX` 过期时间（毫秒）。
  - `NX` 仅新增值。
  - `XX` 仅更新值。
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

- `LPUSH/RPUSH {key} {val} [{val}...]` 推入链表左/右端。返回链表当前长度（包含此次推入的元素）。
- `LRANGE {key} {start} {stop}` 返回指定索引范围内的元素（包含 `{stop}`）。若 `{start}` 大于链表末尾，则返回空表；若 `{stop}` 大于链表末尾，则把它当作链表末尾的索引来处理。负数表示从末尾开始反向索引。
- `LINDEX {key} {idx}` 返回指定索引的元素。负数表示反向索引。若索引越界，则返回 nil。
- `LTRIM {key} {start} {stop}` 只保留范围内的元素（包含 `{stop}`）。
- `LPOP/RPOP {key}` 弹出左/右端的一个元素。当 `{key}` 不存在时返回 nil。

阻塞式命令

- `BLPOP/BRPOP {key} [{key}...] {timeout}` 从第一个非空链表中弹出左/右端元素，若无可弹元素则等待。
- `RPOPLPUSH {srcKey} {destKey}` 弹出列表 A 的右端元素并推入列表 B 的左端，然后返回该元素。
- `BRPOPLPUSH {srcKey} {destKey}` 弹出列表 A 的右端元素并推入列表 B 的左端，然后返回该元素。若无可弹元素则等待。

## SET 集合

- `SADD {key} {member} [{member}...]` 加入元素。返回本次操作真正新增的元素个数。
- `SREM {key} {member} [{member}...]` 删除指定元素。返回本次操作真正删除的元素个数。
- `SMEMBERS {key}` 返回集合中所有元素。
- `SISMEMBER {key} {member}` 集合中是否包含指定元素。若包含返回 1；若不包含或 `{key}` 不存在返回 0。
- `SCARD {key}` 集合包含元素的数量。
- `SRANDMEMBER {key} [count]` 从集合中随机返回一个或多个元素。`count` 为负数时结果可能会重复，反之不会。
- `SPOP {key}` 随机删除一个元素，返回该元素。
- `SMOVE {srcKey} {destKey} {item}` 从集合 A 中删除指定元素并移至集合 B 中。若转移成功则返回 1，否则 0。

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
- `ZREM {key} {member} [{member}...]` 删除指定元素，返回被删除的数量。
- `ZINCBY {key} {inc} {member}` 将指定元素加上指定分值。
- `ZSCORE {key} {member}` 返回指定元素的分值。
- `ZCOUNT {key} {min} {max}` 返回 `{min}` 和 `{max}` 之间的成员数量，包含两端。
- `ZRANK {key} {member}` 返回指定元素在集合中的排名，第一个元素排名 0。
- `ZRANGE {key} {start} {end} [WITHSCORES]` 返回索引范围内的元素。按照分值由低到高排列，相同分值的按照字母顺序排列。
  - `WITHSCORES` 连带返回对应的分值。
- `ZRANGEBYSCORE {key} {min} {max} [WITHSOCRES] [LIMIT {offset} {count}]` 返回分值介于 `{min}` 和 `{max}` 之间的所有成员，默认包含两端。
  - 限定的分值前加上 `(` 表示开区间。如：`ZRANGEBYSCORE zs (1 3`。
  - `{min}` 和 `{max}` 分别可以是 `-inf` 和 `+inf`。
  - `WITHSCORES` 连带返回对应的分值。
  - `LIMIT {offset} {count}` 取指定区间内从 `{offset}` 开始的 `{count}` 个成员。`{count}` 为负数时表示不限制数量。
- `ZREMRANGEBYRANK {key} {start} {end}` 移除排名在指定范围内的成员。
- `ZREMRANGEBYSCORE {key} {min} {max}` 移除分值在指定范围内的成员。
- `ZINTERSTORE {destKey} {keyCount} {key} [{key}...] [WEIGHTS {weight} [{weight}...]] [AGGREGATE SUM|MIN|MAX]` 对指定的若干有序集合求交集，结果存至 `{destKey}`（若该键值已存在则会被覆盖）。结果中各成员的分值默认是各集合中的总和。
  - `{keyCount}` 求交集的集合的数量。
  - `WEIGHTS {weight} [{weight}...]` 指定各集合的分值的权重（用来计算交集中的分值），默认为 1。
  - `AGGREGATE SUM|MIN|MAX` 指定如何计算结果集合中的分值，默认是求和。
- `ZUNIONSTORE {destKey} {keyCount} {key} [{key}...] [WEIGHTS {weight} [{weight}...]] [AGGREGATE SUM|MIN|MAX]` 对指定的若干有序集合求并集，结果存至 `{destKey}`（若该键值已存在则会被覆盖）。结果中各成员的分值默认是各集合中的总和。
  - `{keyCount}` 求并集的集合的数量。
  - `WEIGHTS {weight} [{weight}...]` 指定各集合的分值的权重（用来计算交集中的分值），默认为 1。
  - `AGGREGATE SUM|MIN|MAX` 指定如何计算结果集合中的分值，默认是求和。
- `ZCARD {key}` 返回指定集合的基数。

**部分命令有逆序版本**

- `ZREVRANK`
- `ZREVRANGE`
- `ZREVRANGEBYSCORE`

## 发布和订阅

- `SUBSCRIBE {channel} [{channel}...]` 订阅若干个频道。
- `PSUBSCRIBE {pattern} [{pattern}...]` 订阅与给定模式匹配的所有频道。
- `UNSUBSCRIBE [{channel}...]` 退订若干个频道。若未指定频道，则退订所有频道。
- `PUNSUBSCRIBE [{pattern}...]` 退订与给定模式匹配的频道。若未指定模式，则退订所有频道。
- `PUBLISH {channel} {msg}` 向指定频道发送消息。

## 排序

- `SORT {srcKey} [BY pattern] [LIMIT {offset} {count}] [GET {pattern} [GET {pattern}...]] [ASC|DESC] [ALPHA] [STORE {destKey}]` 对指定列表、集合或有序集合排序，然后返回或存储排序结果。
  - `BY {pattern}` 以外键来排序。将 `{srcKey}` 中的成员替换至 `{pattern}` 中第一个 `*` 号，然后以替换后的 `{pattern}` 键对应的值来排序。可以跟 `GET` 选项配合，指定一个不存在的 `{pattern}`，从而只是做关联查询而不做排序。
    - `BY {pattern}->{field}` 使用 `{pattern}` 对应的散列中的指定字段的值（而不是 `{pattern}` 对应的值）。
  - `LIMIT {offset} {count}` 把结果限制为 `{offset}` 开始的 `{count}` 个成员。
  - `GET {pattern}` 指定返回的结果（而不是排序后的 `{srcKey}` 的值）。跟 `BY {pattern}` 一样的替换方式，返回若干个 `{pattern}` 对应的值。
    - `GET {pattern}->{field}` 同 `BY {pattern}->{field}`。
  - `ASC|DESC` 升序/降序（默认升序）。
  - `ALPHA` 按照字母表顺序而不是数字顺序。
  - `STORE {destKey}` 将排序结果存储至 `{destKey}`，并返回存储的成员数量（而不是排序后的列表）。通常是跟 `EXPIRE` 命令结合，实现排序结果的缓存。

## 过期时间

- `PERSIST {key}` 移除过期时间。
- `TTL {key}` 查看距离过期还有多少秒。
- `PTTL {key}` 查看距离过期还有多少毫秒（>= Redis 2.6）。
- `EXPIRE {key} {seconds}` 在指定秒后过期。
- `EXPIREAT {key} {timeStamp}` 在指定 UNIX 时间点过期。
- `PEXPIREAT {key} {timeStamp}` 在指定毫秒级 UNIX 时间点过期（>= Redis 2.6）。

# 数据安全和性能保障

## 持久化

### 快照持久化

**配置**

```
save 60 1000
stop-writes-on-bgsave-error no
rdbcompression yes
dbfilename dump.rdb # 快照文件名
dir ./ # 快照文件存至此目录下
```

快照保存的数据是截止至快照开始时间点的。快照开始创建到完成创建之间的数据不在其内。

**创建方法**

- 向 Redis 发送 `BGSAVE` 命令。
  Redis 会 fork 一个子进程来处理快照操作。
- 向 Redis 发送 `SAVE` 命令。
  Redis 在快照创建完毕之前不会响应任何命令。通常只在没有足够内存进行 `BGSAVE` 时才这样做。
- 当满足配置的条件时。
  `save 60 1000` 表示“距上次快照超过 60 秒且期间至少有 1000 次写入”。可以配置多个 `save` 选项。任意一个条件满足时都会触发 `BGSAVE`。
- 当 Redis 通过 `SHUTDOWN` 命令接收到关闭服务器请求时，或收到标准 `TERM` 信号时，会执行 `SAVE`，并在完成后关闭服务器。
- 当一台 Redis 服务器连接另一台 Redis 服务器，并向对方发送 `SYNC` 命令来开始一次复制操作时。
  若主服务器目前没有在执行 `BGSAVE` 也不是刚刚执行完 `BGSAVE`，则主服务器会开始 `BGSAVE`。

**性能**

Redis 占用的内存越多，`BGSAVE` 创建子进程所用的时间就越多。`SAVE` 节约了创建子进程的时间，并且由于没有子进程争抢资源，快照会快一点，但是会阻塞来自客户端的指令。

### AOF持久化

> 只追加文件（append-only file）

**配置**

```
appendonly no|yes
appendfsync everysec
no-appendfsync-on-rewrite no
auto-aof-rewrite-percentage 100
auto-aof-rewrite-min-size 64mb
dir ./
```

- `appendfsync`
  - `always` 每个 Redis 写命令都同步写入硬盘。
    系统崩溃时损失最小，但对硬盘性能和固态硬盘寿命有较大损耗。
  - `everysec` 每秒执行一次同步。
    性能损耗很小，系统崩溃时最多损失一秒的数据。硬盘写入忙时 Redis 会自动放慢自己的写入速度。
  - `no` 由操作系统决定何时同步。
    完全不影响 Redis 性能，但系统崩溃时无法确定损失的数据大小。若硬盘写入速度不够快会占满缓冲区，导致 Redis 写入操作被阻塞。

### 重写/压缩AOF文件

- `BGREWRITEAOF` 命令 Redis 创建子进程来移除 AOF 中的冗余命令。
- `auto-aof-rewrite-percentage` `auto-aof-rewrite-min-size` 整两个配置是“且”关系，指定了自动执行 `BGREWRITEAOF` 的条件：当 AOF 文件增大百分比（(当前大小 - 上次大小) / 上次大小）大于某值且 AOF 文件大小大于指定值时。

## 复制

### 配置

```
salveof {host} {port}
```

- `SLAVEOF no one` 命令从服务器停止接受数据更新。
- `SLAVEOF {host} {port}` 命令从服务器开始接受数据更新。

### 复制过程

1. 主：等待命令。
   从：连接主服务器并发送 `SYNC` 命令。
2. 主：开始 `BGSAVE`，并用缓冲区来记录此刻开始接收到的写命令。
   从：根据配置来决定是使用现有数据来响应客户端请求还是向客户端返回错误。
3. 主：`BGSAVE` 完毕，向从服务器发送快照文件，发送期间继续用缓冲区记录写命令。
   从：丢弃原有数据，载入收到的快照文件。
4. 主：快照发送完毕，开始发送缓冲区里的写命令。
   从：快照载入完毕，开始正常接收命令。
5. 主：缓冲区发送完毕，此刻开始每执行一个写命令就发送给从服务器相同的命令。
   从：执行主服务器发来的所有写命令，包括来自缓冲区的和实时的。

**不能配置主主复制，两个互为主服务器的实例会持续占用资源来不断尝试通信，并且客户端从不同实例得到的数据可能不同甚至为空。**

## 处理系统故障

### 验证快照文件和 AOF 文件

- `redis-check-aof --fix {file}` 找出第一个出错的命令并删除此后的所有命令。一般删除的都是 AOF 末尾不完整的写命令。
- `redis-check-dump {file}`

### 更换主服务器

1. 登录从服务器，并打开 `redis-cli`。
2. `SAVE` 保存快照，然后 `QUIT` 退出 `redis-cli`。
3. `scp /path/on/slave/dump.rdb {newMasterIP}:/path/on/master/dump.rdb` 把快照发送至新的主服务器。
4. 登录新的主服务器。
5. `sudo /etc/init.d/redis-server start` 启动 redis。
6. `exit` 退回到从服务器。
7. `redis-cli`
8. `slaveof {newMasterIP} {port}` redis 连接新的主服务器。

## 事务

- `WATCH {key}`: 若从 `WATCH` 到 `EXEC` 的时间内被监视的键发生变化则在执行 `EXEC` 时会返回一个错误。
- `UNWATCH`: 在 `WATCH` 之后、`MULTI` 之前重置连接。
- `MULTI`: 开始事务。
- `EXEC`: 提交事务。
- `DISCARD`: 在 `MULTI` 之后、`EXEC` 之前重置连接（包括取消 `WATCH` 和清空已入队的命令）。

## 流水线

可以用 `MULTI/EXEC` 包裹多个命令来使用流水线从而减少通信次数、提高性能。但事务本身也会消耗资源，并可能导致其他命令被延迟执行。

通常客户端中会有 `pipeline()` 方法，它根据入参来决定是否使用事务流水线。

## 关于性能的其他注意事项

- `redis-benchmark -c 1 -q` 使用 redis 附带的测试工具来测试各命令的性能极限。
  - `-q` 简化输出结果。
  - `-c 1` 使用一个客户端来测试。默认使用 50 个客户端。

不同的实际性能可能对应的问题

|性能或错误                               |可能的原因               |解决方法|
|:----------------------------------------|:------------------------|:-------|
|50%~60%                                  |未使用流水线             |无      |
|25%~30%                                  |每个/组命令都创建了新连接|重用连接|
|返回错误“Cannot assign requested address”|每个/组命令都创建了新连接|重用连接|


# 降低内存占用

## 短结构

redis 会将体积较小的数据用压缩列表的结构存储，从而降低内存占用。具体的阈值配置如下。

对于体积较大的压缩列表进行操作反而会降低性能，因此要选取合适的阈值。

若整数集合的所有成员都可解释为十进制整数，且都在平台的有符号整数范围内，且成员数量少于阈值，那么会以有序整数数组的方式存储。

```
 # 链表
list-max-ziplist-entries 512 # 压缩列表允许包含的最大元素数量
list-max-ziplist-value 64 # 压缩列表每个节点的最大体积（字节）

 # 散列
hash-max-ziplist-entries 512
hash-max-ziplist-value 64

 # 有序集合
zset-max-ziplist-entries 128
zset-max-ziplist-value 64

 # 整数集合
set-max-intset-entries 512
```

同种数据结构的限制条件中任一被突破时，redis 会把压缩列表转换为其他结构，占用的内存也相应增加。

# 集群

## 部署

1. 准备目录结构：`~/redis/conf`、`~/redis/data`、`~/redis/log`。
2. 准备各节点的配置文件：`~/redis/conf/master-6379.conf` 以及 6380、6381。
3. 配置中至少有以下配置项：
```
port 6379
cluster-enabled yes
cluster-node-timeout 15000
cluater-config-file "nodes-6379.conf"
 # 还有其他单机模式的配置项
```
4. 启动所有节点：`redis-server conf/redis-{port}.conf`
```sh
cd ./conf
redis-server ./master-6379.conf &
redis-server ./master-6380.conf &
redis-server ./master-6381.conf &
```
5. 检查日志：`cat log/redis-{port}.log`
6. 客户端登录：`redis-cli -p 6379`
7. 各节点握手：`cluster meet 127.0.0.1 6380`（如果要从其他主机访问集群的话，这里的 IP 要写 IPv4 地址）
8. 分配槽（在 linux 命令行执行而不是 redis-cli 中）：`redis-cli -h 127.0.0.1 -p 6379 cluster addslots {0..5460}`，尽量等分的话是 0..5460、5461..10921、10922..16383。
9. 可以使用了。
10. 关闭所有节点
```sh
 #!/bin/bash
 # 正常关闭
redis-cli -p 6379 shutdown nosave
redis-cli -p 6380 shutdown nosave
redis-cli -p 6381 shutdown nosave
```

## 节点

redis 服务器启动时根据配置项 `cluster-enabled=yes|no` 来决定成为集群的一个节点还是处于单机模式的一个普通服务器。

- `cluster nodes` 查看集群中的节点。
- `cluster meet {ip} {port}` 令当前节点与指定节点握手。
- `cluster info` 查看节点状态信息。
- `cluster addslots slot [slot...]` 向当前节点分配槽。只能一个一个槽地写，不能用通配符或范围符号之类的。
- `cluster delslots slot [slot...]` 删除槽。
- `cluster keyslot {key}` 计算键属于哪个槽。
- `cluster slots` 查看节点与槽的映射关系。

# 客户端

## 客户端通信协议

1. 通信协议在 TCP 协议之上构建。
2. Redis 制订了 RESP （REdis Serialization Protocol）这种简单高效的协议。

**发送命令格式**

```
*{参数数量} CRLF
${参数1的字节数量} CRLF
{参数1} CRLF
...
${参数N的字节数量} CRLF
{参数N} CRLF
```

实际传输的格式类似：

```
*3\r\n$3\r\nSET\r\n$5\r\nhello\r\n
```

**返回结果格式**

- 状态回复：第一个字节为 `+`。
- 错误回复：第一个字节为 `-`。
- 整数回复：第一个字节为 `:`。
- 字符串回复：第一个字节为 `$`。
- 多条字符串回复：第一个字节为 `*`。

