# 1 前言
参考文章：

Redis概念详细介绍    https://blog.csdn.net/eroswang/article/details/7080412

Redis核心原理       https://www.jianshu.com/p/4e6b7809e10a


**RE**mote **DI**ctionary **S**erver(Redis) 是一个由Salvatore Sanfilippo写的key-value存储系统。Redis提供了一些丰富的数据结构，包括 lists、sets、ordered sets 、hashes，当然还有和Memcached一样的 strings结构。Redis当然还包括了对这些数据结构的丰富操作。

## 1.1 优点

* 性能极高 – Redis能支持超过 100K+ 每秒的读写频率。（可参考另外的文章，redis：单线程实现百万QPS，主要是三个方面，数据结构、多路复用、事务模型）
* 丰富的数据类型 – Redis支持二进制案例的 Strings, Lists, Hashes, Sets 及 Ordered Sets 数据类型操作。
* 原子 – Redis的所有操作都是原子性的，同时Redis还支持对几个操作全并后的原子性执行。
* 丰富的特性 – Redis还支持 publish/subscribe, 通知, key 过期等等特性。


## 1.2 作用

常规业务系统的数据库访问中，读写操作的比例一般在7/3到9/1，也就是说读操作远多于写操作，因此高并发系统设计里，通过NoSQL技术将热点数据（短期内变动概率小的数据）放入内存以达到减轻DB压力，提升数据访问速度的目的，Redis和MongoDB是当下应用最广泛的NoSQL产品。

当然如果系统里的写操作居多，也没有必要使用缓存。

因此Redis主要用于：

1. 解决**访问性能和并发能力**的问题。

2. 除了纯数据缓存的作用之外，得益于其超高速的响应能力，Redis也常用于**提供分布式锁的解决方案**。

# 2 数据类型 使用介绍
5种数据类型：Strings, Lists, Hashes, Sets 及 Ordered Sets

## 2.1 String类型
Redis能存储二进制安全的字符串，最大长度为1GB：

```bash
redis 127.0.0.1:6379> SET name "John Doe"
OK
redis 127.0.0.1:6379> GET name
"John Doe"
```

String类型还支持批量的读写操作：

```bash
# multi set，简称 MSET 批量设置
redis 127.0.0.1:6379> MSET age 30 sex "male"
OK

# multi get，简称 MGET 批量查询
redis 127.0.0.1:6379> MGET age sex

# set的时候用的30，get的时候是字符串"30"，和bash里面一样，字面量
1) "30"
2) "male"
```

String类型的纯数字字符串，其实也可以被当作数字来处理，从而支持对数字的加减操作（和bash一样）。

```bash
# INCR 命令将 key 中储存的数字值增1。
# 如果 key 不存在，那么 key 的值会先被初始化为 0 ，然后再执行 INCR 操作。
# 如果值包含错误的类型，或字符串类型的值不能表示为数字，那么返回一个错误。
redis 127.0.0.1:6379> INCR age
(integer) 31

redis 127.0.0.1:6379> INCRBY age 4
(integer) 35

redis 127.0.0.1:6379> GET age
"35"

redis 127.0.0.1:6379> DECR age
(integer) 34

redis 127.0.0.1:6379> DECRBY age 4
(integer) 30

redis 127.0.0.1:6379> GET age
"30"
```

String类型还支持对其部分的修改和获取操作：

```bash
# 追加操作，返回追加之后的，字符串的总长度
redis 127.0.0.1:6379> APPEND name " Mr."
(integer) 12
redis 127.0.0.1:6379> GET name
"John Doe Mr."

# 字符串类型的常用命令
redis 127.0.0.1:6379> STRLEN name
(integer) 12

# redis的索引不是左闭右开的！！！是[index1,index2]，和一般的编程语言不同！！！
redis 127.0.0.1:6379> SUBSTR name 0 3
"John"
```

## 2.2 List类型
Redis能够将数据存储成一个链表，并能对这个链表进行丰富的操作。**针对List的操作命令都是以L开头的**：

```bash
redis 127.0.0.1:6379> LPUSH students "John Doe"
(integer) 1
redis 127.0.0.1:6379> LPUSH students "Captain Kirk"
(integer) 2
redis 127.0.0.1:6379> LPUSH students "Sheldon Cooper"
(integer) 3

redis 127.0.0.1:6379> LLEN students
(integer) 3

redis 127.0.0.1:6379> LRANGE students 0 2
1) "Sheldon Cooper"
2) "Captain Kirk"
3) "John Doe"

redis 127.0.0.1:6379> LPOP students
"Sheldon Cooper"

redis 127.0.0.1:6379> LLEN students
(integer) 2

redis 127.0.0.1:6379> LRANGE students 0 1
1) "Captain Kirk"
2) "John Doe"

# Lrem 根据参数 COUNT 的值，移除列表中与参数 VALUE 相等的元素。
# COUNT 的值可以是以下几种：
# count > 0 : 从表头开始向表尾搜索，移除与 VALUE 相等的元素，数量为 COUNT 。
# count < 0 : 从表尾开始向表头搜索，移除与 VALUE 相等的元素，数量为 COUNT 的绝对值。
# count = 0 : 移除表中所有与 VALUE 相等的值。
redis 127.0.0.1:6379> LREM students 1 "John Doe"
(integer) 1

redis 127.0.0.1:6379> LLEN students
(integer) 1

redis 127.0.0.1:6379> LRANGE students 0 0
1) "Captain Kirk"
```

Redis也支持很多修改操作：

```bash
redis 127.0.0.1:6379> LINSERT students BEFORE "Captain Kirk" "Dexter Morgan"
(integer) 3

redis 127.0.0.1:6379> LRANGE students 0 2
1) "Dexter Morgan"
2) "Captain Kirk"
3) "John Doe"

redis 127.0.0.1:6379> LPUSH students "Peter Parker"
(integer) 4

redis 127.0.0.1:6379> LRANGE students 0 3
1) "Peter Parker"
2) "Dexter Morgan"
3) "Captain Kirk"
4) "John Doe"

redis 127.0.0.1:6379> LTRIM students 1 3
OK

redis 127.0.0.1:6379> LLEN students
(integer) 3

redis 127.0.0.1:6379> LRANGE students 0 2
1) "Dexter Morgan"
2) "Captain Kirk"
3) "John Doe"

redis 127.0.0.1:6379> LREM students 1 "John Doe"
(integer) 1

redis 127.0.0.1:6379> LLEN students
(integer) 1

redis 127.0.0.1:6379> LRANGE students 0 1
1) "Captain Kirk"
```

## 2.3 Set类型
Redis能够将一系列不重复的值存储成一个集合，针对Set数据类型的操作以S开头：

```bash
redis 127.0.0.1:6379> SADD birds crow
(integer) 1

redis 127.0.0.1:6379> SADD birds pigeon
(integer) 1

redis 127.0.0.1:6379> SADD birds bat
(integer) 1

redis 127.0.0.1:6379> SADD mammals dog
(integer) 1

redis 127.0.0.1:6379> SADD mammals cat
(integer) 1

redis 127.0.0.1:6379> SADD mammals bat
(integer) 1

redis 127.0.0.1:6379> SMEMBERS birds
1) "bat"
2) "crow"
3) "pigeon"

redis 127.0.0.1:6379> SMEMBERS mammals
1) "bat"
2) "cat"
3) "dog"
```

Sets结构也支持相应的修改操作：

```bash
redis 127.0.0.1:6379> SREM mammals cat
(integer) 1

redis 127.0.0.1:6379> SMEMBERS mammals
1) "bat"
2) "dog"

redis 127.0.0.1:6379> SADD mammals human
(integer) 1

redis 127.0.0.1:6379> SMEMBERS mammals
1) "bat"
2) "human"
3) "dog"
```

Redis还支持对集合的子交并补等操作

```bash
# 取交集
redis 127.0.0.1:6379> SINTER birds mammals
1) "bat"

# 取补集
redis 127.0.0.1:6379> SUNION birds mammals
1) "crow"
2) "bat"
3) "human"
4) "pigeon"
5) "dog"

# 取差集，即 第一个集合独一份的元素
redis 127.0.0.1:6379> SDIFF birds mammals
1) "crow"
2) "pigeon"
```

## 2.4 Sorted Set有序集合
Sorted Sets和Sets结构相似，不同的是存在Sorted Sets中的数据会有一个甚至多个score属性，并会在写入时就按这个score排好序。针对该数据结构的命令，以Z开头。

```bash
# ZADD KEY_NAME SCORE1 VALUE1.. SCORE_N VALUE_N  
# 可一次添加一个元素，也可以一次添加多个元素
# 分数值可以是整数值或双精度浮点数。
redis 127.0.0.1:6379> ZADD days 0 mon
(integer) 1

redis 127.0.0.1:6379> ZADD days 1 tue
(integer) 1

redis 127.0.0.1:6379> ZADD days 2 wed
(integer) 1

redis 127.0.0.1:6379> ZADD days 3 thu
(integer) 1
redis 127.0.0.1:6379> ZADD days 4 fri
(integer) 1
redis 127.0.0.1:6379> ZADD days 5 sat
(integer) 1
redis 127.0.0.1:6379> ZADD days 6 sun
(integer) 1

redis 127.0.0.1:6379> ZCARD days
(integer) 7

redis 127.0.0.1:6379> ZRANGE days 0 6
1) "mon"
2) "tue"
3) "wed"
4) "thu"
5) "fri"
6) "sat"
7) "sun"

redis 127.0.0.1:6379> ZSCORE days sat
"5"

# redis是[  ]的。count用于[score1,score2]之间的计数
redis 127.0.0.1:6379> ZCOUNT days 3 6
(integer) 4

redis 127.0.0.1:6379> ZRANGEBYSCORE days 3 6
1) "thu"
2) "fri"
3) "sat"
4) "sun"

```

## 2.5 Hash
Redis能够存储key对多个属性的数据（比如user1.uname user1.passwd）

```bash
redis 127.0.0.1:6379> HKEYS student
1) "name"
2) "age"
3) "sex"

redis 127.0.0.1:6379> HVALS student
1) "Ganesh"
2) "30"
3) "Male"

redis 127.0.0.1:6379> HGETALL student
1) "name"
2) "Ganesh"
3) "age"
4) "30"
5) "sex"
6) "Male"

redis 127.0.0.1:6379> HDEL student sex
(integer) 1

redis 127.0.0.1:6379> HGETALL student
1) "name"
2) "Ganesh"
3) "age"
4) "30"
```

Hash数据结构能够批量修改和获取：

```bash
redis 127.0.0.1:6379> HMSET kid name Akshi age 2 sex Female
OK
redis 127.0.0.1:6379> HMGET kid name age sex
1) "Akshi"
2) "2"
3) "Female"
```
# 3 发布/订阅 使用介绍
Redis支持这样一种特性，你可以将数据推到某个信息管道/key中，然后其它人可以通过订阅这些管道来获取推送/更新过来的信息。

## 3.1 发布/订阅单个

客户端1订阅一个键：

```bash
# 控制台打印输出了一些反馈信息，然后终端进入独占状态
redis 127.0.0.1:6379> SUBSCRIBE channelone
Reading messages... (press Ctrl-C to quit)
1) "subscribe"
2) "channelone"
3) (integer) 1
```

客户端2往这个管道推送信息：

```bash
redis 127.0.0.1:6379> PUBLISH channelone hello
(integer) 1
redis 127.0.0.1:6379> PUBLISH channelone world
(integer) 1
```

然后客户端1就能获取到推送的信息：

```bash
redis 127.0.0.1:6379> SUBSCRIBE channelone
Reading messages... (press Ctrl-C to quit)
1) "subscribe"
2) "channelone"
3) (integer) 1
# 以上是订阅时的反馈信息

# 客户端2第一次发布触发的订阅更新
1) "message"
2) "channelone"
3) "hello"

# 客户端2第二次发布触发的订阅更新
1) "message"
2) "channelone"
3) "world"
```
## 3.2 发布/订阅 多个

用下面的命令订阅所有channel开头的键：

```bash
# PSUBSCRIBE  批量订阅
redis 127.0.0.1:6379> PSUBSCRIBE channel*
Reading messages... (press Ctrl-C to quit)
1) "psubscribe"
2) "channel*"
3) (integer) 1
```

在另一个客户端对两个推送信息：

```bash
redis 127.0.0.1:6379> PUBLISH channelone hello
(integer) 1

redis 127.0.0.1:6379> PUBLISH channeltwo world
(integer) 1
```

然后在第一个客户端就能收到推送的信息：

```bash
redis 127.0.0.1:6379> PSUBSCRIBE channel*
Reading messages... (press Ctrl-C to quit)
1) "psubscribe"
2) "channel*"
3) (integer) 1
# 以上是订阅反馈

# 比普通的订阅更新多一行描述信息
1) "pmessage"
2) "channel*"
3) "channelone"
4) "hello"

1) "pmessage"
2) "channel*"
3) "channeltwo"
4) "world"
```

# 4 数据过期设置

Redis支持按key设置过期时间，过期后值将被删除（**仅仅是值被删除了，通过TTL key_name还是能查到该键已过期**，彻底删除需要使用Del）。

```bash
EXPIRE <KEY> <TTL> : 将键的生存时间设为 ttl 秒
PEXPIRE <KEY> <TTL> :将键的生存时间设为 ttl 毫秒
EXPIREAT <KEY> <timestamp> :将键的过期时间设为 timestamp 所指定的秒数时间戳
PEXPIREAT <KEY> <timestamp>: 将键的过期时间设为 timestamp 所指定的毫秒数时间戳.
```

用TTL命令可以获取某个key值的过期时间（-1表示永不过期）：

```bash
redis 127.0.0.1:6379> SET name "John Doe"
OK

redis 127.0.0.1:6379> TTL name
(integer) -1
```

下面命令先用EXISTS命令查看key值是否存在，然后设置了5秒的过期时间：

```bash
redis 127.0.0.1:6379> SET name "John Doe"
OK

redis 127.0.0.1:6379> EXISTS name
(integer) 1

redis 127.0.0.1:6379> EXPIRE name 5
(integer) 1
```

5秒后再查看：

```bash
# 已过期，0表示不存在
redis 127.0.0.1:6379> EXISTS name
(integer) 0

redis 127.0.0.1:6379> GET name
(nil)
```

上面在是直接设置多少秒后过期，redis也可以设置在某个时间戳过期，下面例子是设置2011-09-24 00:40:00过期。

```bash
redis 127.0.0.1:6379> SET name "John Doe"
OK

redis 127.0.0.1:6379> EXPIREAT name 1316805000
(integer) 1

redis 127.0.0.1:6379> EXISTS name
(integer) 0
```

redis 的 键 的过期有独特的清理策略：立即删除、懒删除、定期删除，详见 https://www.jianshu.com/p/9352d20fb2e0


对有过期时间的键进行操作，对其生命周期的影响：

http://www.redis.cn/commands/expire.html

https://zhuanlan.zhihu.com/p/54758076

# 5 事务性
Redis本身支持一些简单的组合型的命令，比如以NX结尾命令都是判断在这个值没有时才进行某个命令。

```bash
redis 127.0.0.1:6379> SET name "John Doe"
OK

# 操作不成功，返回0
redis 127.0.0.1:6379> SETNX name "Dexter Morgan"
(integer) 0

redis 127.0.0.1:6379> GET name
"John Doe"

# 先返回旧值，再设置新值
redis 127.0.0.1:6379> GETSET name "Dexter Morgan"
"John Doe"

redis 127.0.0.1:6379> GET name
"Dexter Morgan"
```

当然，Redis还支持自定义的命令组合，通过MULTI和EXEC，将几个命令组合起来执行：

```bash
redis 127.0.0.1:6379> SET counter 0
OK

# 组合命令/事务 的开始标志
redis 127.0.0.1:6379> MULTI
OK

# 不会立刻执行
redis 127.0.0.1:6379> INCR counter
QUEUED

redis 127.0.0.1:6379> INCR counter
QUEUED

redis 127.0.0.1:6379> INCR counter
QUEUED

# 事务结束标志，EXEC发出之后，才开始按序一起执行
redis 127.0.0.1:6379> EXEC
1) (integer) 1
2) (integer) 2
3) (integer) 3

redis 127.0.0.1:6379> GET counter
"3"
```

可以直接用DICARD命令来中断执行中的命令序列：

```bash
redis 127.0.0.1:6379> SET newcounter 0
OK

redis 127.0.0.1:6379> MULTI
OK

redis 127.0.0.1:6379> INCR newcounter
QUEUED

redis 127.0.0.1:6379> INCR newcounter
QUEUED

redis 127.0.0.1:6379> INCR newcounter
QUEUED

#声明事务中断
redis 127.0.0.1:6379> DISCARD
OK

redis 127.0.0.1:6379> GET newcounter
"0"
```

# 6 持久化
Redis的所有运行数据都存储在内存中，但是他也提供对这些数据的持久化。分为三中方式：
* 快照持久化，rdb
* 追加式记录 aof
* 两种混合

## 6.1 数据快照
数据快照的原理是将整个Redis中存的**所有数据遍历一遍**存到一个扩展名为rdb的数据文件中。通过SAVE命令可以调用这个过程。

```bash
redis 127.0.0.1:6379> SET name "John Doe"
OK

redis 127.0.0.1:6379> SAVE
OK

redis 127.0.0.1:6379> SET name "Sheldon Cooper"
OK

redis 127.0.0.1:6379> BGSAVE
Background saving started
```

但是，可以对 Redis 进行设置， 让它在N秒内数据集至少有M个改动这一条件被满足时， 自动保存一次数据集。

比如说， 以下设置会让 Redis 在满足60秒内有至少有1000个键被改动”这一条件时， 自动保存一次数据集：

```bash
save 60 1000
# redis 配置文件种默认rdb配置有：
# save 900 1
# save 300 10
# save 60 10000
```


## 6.2 Append-Only File（追加式的操作日志记录）
Redis还支持一种追加式的操作日志记录，叫append only file，其日志文件以aof后缀结尾。这种将修改的每一条指令记录进文件要开启aof日志的记录，需要手动在配置文件中进行如下设置：

```
appendonly yes
```

这时候你所有的操作都会记录在aof日志文件中

```bash
redis 127.0.0.1:6379> GET name
(nil)

redis 127.0.0.1:6379> SET name "Ganesh Gunasegaran"
OK

redis 127.0.0.1:6379> EXIT
```

aof日志文件如下：

```
→ cat /usr/local/var/db/redis/appendonly.aof
*2
$6
SELECT
$1
0
*3
$3
SET
$4
name
$18
Ganesh Gunasegaran
```

可以配置 Redis 多久才将数据fsync（force sync，fsync() 把文件数据和文件元信息写入强刷到磁盘中，速度是比较慢的，文件是在内存中操作的，操作完让内核回写到磁盘）到磁盘一次。

redis的fsync频率策略有三种：
* 每次有新命令追加到 AOF 文件时就执行一次fsync：非常慢，也非常安全。
* 每秒fsync一次：足够快（和使用 RDB 持久化差不多），并且在故障时只会丢失 1 秒钟的数据。
* 从不fsync：将数据交给操作系统来处理。更快，也更不安全的选择。

## 6.3 Redis 4.0 混合型持久化

重启 Redis 时，我们很少使用 rdb 来恢复内存状态，因为会丢失大量数据。我们通常使用 AOF 日志重放，但是**重放 AOF 日志性能相对 rdb 来说要慢很多**，这样在 Redis 实例很大的情况下，启动需要花费很长的时间。 Redis 4.0 为了解决这个问题，带来了一个新的持久化选项——混合持久化。

AOF在重写(aof文件里可能有太多没用指令，所以aof会定期根据内存的最新数据生成aof文件)时，将：

* 重写这一刻之前的内存rdb快照文件的内容
* 增量的 AOF修改内存数据的命令日志文件

存在一起，都写入新的aof文件，新的文件一开始不叫appendonly.aof，等到重写完新的AOF文件才会进行改名，原子的覆盖原有的AOF文件，完成新旧两个AOF文件的替换；

aof 根据配置规则在后台自动重写，也可以人为执行命令bgrewriteaof重写AOF。 于是在 Redis 重启的时候，可以先加载 rdb 的内容，然后再重放增量 AOF 日志就可以完全替代之前的 AOF 全量文件重放，重启效率因此大幅得到提升。

![redis混合持久aof文件结构](./resources/redis混合持久aof文件结构.webp)


# 7 管理命令
Redis支持多个DB，默认是16个，你可以设置将数据存在哪一个DB中，不同DB间的数据具有隔离性。也可以在多个DB间移动数据。

```bash
# 选取第一个DB为操作数据库对象
redis 127.0.0.1:6379> SELECT 0
OK

redis 127.0.0.1:6379> SET name "John Doe"
OK

# 选取第二个DB为操作数据库对象
redis 127.0.0.1:6379> SELECT 1
OK

redis 127.0.0.1:6379[1]> GET name
(nil)

redis 127.0.0.1:6379[1]> SELECT 0
OK

# 将 键 从当前数据库 移动到 DB 1
redis 127.0.0.1:6379> MOVE name 1
(integer) 1

redis 127.0.0.1:6379> SELECT 1
OK

redis 127.0.0.1:6379[1]> GET name
"John Doe"
```
Redis还能进行一些如下操作，获取一些运行信息：

```bash
redis 127.0.0.1:6379[1]> DBSIZE
(integer) 1

redis 127.0.0.1:6379[1]> INFO
redis_version:2.2.13
redis_git_sha1:00000000
redis_git_dirty:0
arch_bits:64
multiplexing_api:kqueue
```

Redis还支持对某个DB数据进行清除（当然清空所有DB的数据的操作也是支持的）

```bash
redis 127.0.0.1:6379> SET name "John Doe"
OK

# 当前DB中 键 的个数
redis 127.0.0.1:6379> DBSIZE
(integer) 1

redis 127.0.0.1:6379> SELECT 1
OK

redis 127.0.0.1:6379[1]> SET name "Sheldon Cooper"
OK

redis 127.0.0.1:6379[1]> DBSIZE
(integer) 1

redis 127.0.0.1:6379[1]> SELECT 0
OK

# 清空当前DB
redis 127.0.0.1:6379> FLUSHDB
OK

redis 127.0.0.1:6379> DBSIZE
(integer) 0

redis 127.0.0.1:6379> SELECT 1
OK

redis 127.0.0.1:6379[1]> DBSIZE
(integer) 1

# 清空所有DB
redis 127.0.0.1:6379[1]> FLUSHALL
OK

redis 127.0.0.1:6379[1]> DBSIZE
(integer) 0
```

# 8 缓存淘汰策略

当 Redis 内存超出物理内存限制时，内存的数据会开始和磁盘产生频繁的交换 (swap)。交换会让 Redis 的性能急剧下降，对于访问量比较频繁的 Redis 来说，这样龟速的存取效率基本上等于不可用。

在生产环境中我们是不允许 Redis 出现交换行为的，为了限制最大使用内存，Redis 提供了配置参数 maxmemory 来限制redis的最大使用内存大小。

当实际内存超出 maxmemory 时，Redis 提供了几种可选策略 (maxmemory-policy) 来让用户自己决定该如何腾出新的空间以继续提供读写服务。

* noeviction：

不会继续服务写请求 (DEL 请求可以继续服务)，读请求可以继续进行。这样可以保证不会丢失数据，但是会让线上的业务不能持续进行。这是默认的淘汰策略。


**设置了过期时间的key是volatile的。**


* volatile-lru（淘汰策略是LRU，least recently used）：

尝试淘汰设置了过期时间的 key，最少使用的 key 优先被淘汰。没有设置过期时间的 key 不会被淘汰，这样可以保证需要持久化的数据不会突然丢失。

* volatile-ttl：

跟上面一样，除了淘汰的策略不是 LRU，而是 key 的剩余寿命 ttl 的值，ttl 越小越优先被淘汰。

* volatile-random：

跟上面一样，不过淘汰的 key 是过期 key 集合中随机的 key。

* allkeys-lru：

区别于 volatile-lru，这个策略要淘汰的 key 对象是全体的 key 集合，而不只是过期的 key 集合。这意味着没有设置过期时间的 key 也会被淘汰。

* allkeys-random：

跟上面一样，不过淘汰的策略是随机的 key。

* volatile-xxxxxxxh和allkey-xxxxxxx ：

volatile-xxxxxxx策略，表明当前策略只会针对带过期时间的 key 进行淘汰；allkeys-xxxxxxx 策略，表明会对所有的 key 进行淘汰。如果你只是拿 Redis 做缓存，那应该使用 allkeys-xxx，客户端写缓存时不必携带过期时间。如果你还想同时使用 Redis 的持久化功能，那就使用 volatile-xxx 策略，这样可以保留没有设置过期时间的 key，它们是永久的 key 不会被 LRU 算法淘汰。
