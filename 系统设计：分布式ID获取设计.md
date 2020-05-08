# 1 前言
9种 分布式ID生成方式    https://zhuanlan.zhihu.com/p/107420326

# 2 是什么以及为什么分布式ID
在说分布式ID的具体实现之前，我们来简单分析一下为什么用分布式ID？分布式ID应该满足哪些特征？

## 2.1 什么是分布式ID
拿MySQL数据库举个栗子：

在我们业务数据量不大的时候，单库单表完全可以支撑现有业务，数据再大一点搞个MySQL主从同步读写分离也能对付。

但随着数据日渐增长，主从同步也扛不住了，就需要对数据库进行**分库分表**，但分库分表后需要有一个唯一ID来标识一条数据，分裤分表之后的数据库的自增ID显然不能满足需求，多个库之间的自增ID有可能会有重复ID；特别一点的如订单、优惠券也都需要有唯一ID做标识。此时一个能够生成全局唯一ID的系统是非常必要的。那么这个全局唯一ID就叫分布式ID。

## 2.2 那么分布式ID需要满足那些条件
* 全局唯一：必须保证ID是全局性唯一的，基本要求
* 高性能：高可用低延时，ID生成响应要块，否则反倒会成为业务瓶颈
* 高可用：100%的可用性是骗人的，但是也要无限接近于100%的可用性
* 好接入：要秉着拿来即用的设计原则，在系统设计和实现上要尽可能的简单
* 趋势递增：最好**趋势递增**，这个要求就得看具体业务场景了，一般不严格要求

# 3 9种分布式ID生成方式

以下9种，分布式ID生成器方式以及优缺点：
* UUID
* 数据库自增ID
* 数据库多主模式
* 号段模式
* Redis
* 雪花算法（SnowFlake）
* 滴滴出品（TinyID）
* 百度 （Uidgenerator）
* 美团（Leaf）

## 3.1 UUID
在Java的世界里，想要得到一个具有唯一性的ID，首先被想到可能就是UUID，毕竟它有着全球唯一的特性。那么UUID可以做分布式ID吗？答案是可以的，但是并不推荐！

```java
public static void main(String[] args) {        
       String uuid = UUID.randomUUID().toString().replaceAll("-","");       
       System.out.println(uuid); 
}
```

UUID的生成简单到只有一行代码，输出结果 c2b8c2b9e46c47e3b30dca3b0d447718，但UUID却并不适用于实际的业务需求。如果想用作订单号，UUID这样的字符串没有丝毫的意义，看不出和订单相关的有用信息；想作为数据库的字段值，甚至索引/主键，它不仅是太长还是字符串，**存储性能差查询也很耗时**，所以不推荐用作分布式ID。

优点：

生成足够简单，本地生成无网络消耗，具有唯一性

缺点：

* 无序的字符串，不具备趋势自增特性
* 没有具体的业务含义
* 长度过长16 字节，128位，36位长度的字符串，存储以及查询对MySQL的性能消耗较大，**MySQL官方明确建议主键要尽量越短越好，作为数据库主键 UUID 的无序性会导致数据位置频繁变动，严重影响性能**。

## 3.2 数据库自增ID
基于数据库的auto_increment自增ID完全可以充当分布式ID，具体实现：需要一个单独的MySQL实例用来生成ID，建表结构如下：

```sql
CREATE DATABASE `SEQ_ID`;

CREATE TABLE SEQID.SEQUENCE_ID (    
    id bigint(20) unsigned NOT NULL auto_increment,     
    value char(10) NOT NULL default '',    
    PRIMARY KEY (id)
) ENGINE=MyISAM;

insert into SEQUENCE_ID (value)  VALUES ('values');
```

优点：

实现简单，ID单调自增，数值类型查询速度快

缺点：
* DB单点存在宕机风险，无法扛住高并发场景
* 当我们需要一个ID的时候，向表中插入一条记录返回主键ID，但这种方式有一个比较致命的缺点，访问量激增时MySQL本身就是系统的瓶颈，用它来实现分布式服务风险比较大，不推荐！

## 3.3 数据库集群模式

前边说了单点数据库方式不可取，那对上边的方式做一些高可用优化，换成**主从模式集群**。害怕一个主节点挂掉没法用，那就做双主模式集群，也就是两个Mysql实例都能单独的生产自增ID。那这样还会有个问题，两个MySQL实例的自增ID都从1开始，会生成重复的ID怎么办？

解决方案：设置起始值和自增步长

MySQL_1 配置：

```sql
set @@auto_increment_offset = 1;     -- 起始值
set @@auto_increment_increment = 2;  -- 步长
```

MySQL_2 配置：

```sql
set @@auto_increment_offset = 2;     -- 起始值
set @@auto_increment_increment = 2;  -- 步长
```

这样两个MySQL实例的自增ID分别就是：1、3、5、7、9 2、4、6、8、10

那如果集群后的性能还是扛不住高并发咋办？就要进行MySQL扩容增加节点，这是一个比较麻烦的事。水平扩展的数据库集群，有利于解决数据库单点压力的问题，同时为了ID生成特性，将自增步长按照机器数量来设置。

增加第三台MySQL实例需要人工修改一、二两台MySQL实例的起始值和步长，同时，把**第三台机器的ID起始生成位置设定在比现有最大自增ID的位置远一些**，但必须在一、二两台MySQL实例ID还没有增长到第三台MySQL实例的起始ID值的时候，否则自增ID就要出现重复了，必要时可能还需要停机修改。

优点：

解决DB单点问题

缺点：

不利于后续扩容，而且实际上单个数据库自身压力还是大，依旧无法满足高并发场景。




## 3.4 号段模式

号段模式是当下分布式ID生成器的主流实现方式之一。

号段模式可以理解为从数据库批量的获取自增ID，每次从数据库取出一个号段范围，例如 (1,1000] 代表1000个ID，具体的业务服务将本号段，生成1~1000的自增ID并加载到内存。表结构如下：

```sql
CREATE TABLE id_generator (  
id int(10) NOT NULL,  
max_id bigint(20) NOT NULL COMMENT '当前最大的可用id',  
step int(20) NOT NULL COMMENT '号段的步长',  
biz_type int(20) NOT NULL COMMENT '业务类型，代表不同的业务',  
version int(20) NOT NULL COMMENT '版本号',  
PRIMARY KEY (`id`));
```

biz_type ：代表不同业务类型

max_id ：当前最大的可用id

step ：代表号段的长度

version ：是一个乐观锁，每次都更新version，保证并发时数据的正确性

id biz_type max_id step version 1 101 1000 2000 0

等这批号段ID用完，再次向数据库申请新号段，对max_id字段做一次update操作，update max_id= max_id + step，update成功则说明新号段获取成功，新的号段范围是(max_id ,max_id +step]。

```sql
update id_generator set max_id = max_id+step;
version = version + 1 where version = version(上一次号段的版本号) and biz_type = XXX
```

由于同一业务可能同时进行号段申请，所以同一业务类型，采用版本号version乐观锁方式更新，这种分布式ID生成方式不强依赖于数据库，不会频繁的访问数据库，对数据库的压力小很多。
