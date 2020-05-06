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

