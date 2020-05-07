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
redis 127.0.0.1:6379> EXISTS name
(integer) 0

redis 127.0.0.1:6379> GET name
(nil)
# 这个值已经没有了。键名还是在的。
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

