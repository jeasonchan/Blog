# 1 背景

参考文档：

MongoDB Sharded cluster架构原理  https://developer.aliyun.com/article/32434


please create an index that starts with the shard key before sharding    https://www.cnblogs.com/lazyboy/archive/2012/11/26/2789401.html

# 2 为什么要使用Sharded cluster
MongoDB目前3大核心优势：
* 灵活模式，通过json文档来实现灵活模式
* 高可用性，通过复制集来保证高可用
* 可扩展性，通过Sharded cluster来保证可扩展性

当MongoDB复制集遇到下面的业务场景时，你就需要考虑使用Sharded cluster：

* 存储容量需求超出单机磁盘容量
* 活跃的数据集超出单机内存容量，导致很多请求都要从磁盘读取数据，影响性能
* 写IOPS超出单个MongoDB节点的写服务能力

![集合数据分片示意图]](https://yqfile.alicdn.com/4eed80bbaa26f79ad12a74e9113c744d5e77ccf8.jpeg)

如上图所示，Sharding Cluster使得**集合的数据**可以分散到多个Shard（复制集或者单个Mongod节点）存储，使得MongoDB具备了横向扩展（Scale out）的能力，丰富了MongoDB的应用场景。

# 3 Sharded cluster架构
Sharded cluster由Shard、Mongos和Config server 3个组件构成：





# 8 部署
简要步骤
1. 先搭建n个副本集，当做n个分片使用
2. 配置一下配置文件，router、config、n个副本集都正常运行起来
3. 在router使用mongo连接上，执行 sh.addShard("副本集名称/副本集的MongoDB连接串") 
4. 必须先对某个db开启使用分片，sh.enableSharding("db_name")
5. 再对允许分片的db中的某个集合开启分片，sh.shardCollection("db_name.db_collection", {column_name: 1})，其中column_name必须是某索引的第一列，空集合会自动先根据分片列创建索引，非空集合则需要我们手动为column_name先创建一个索引


关于设置账号密码，两种思路：
1. 一开始都不要设置设置账号密码+无鉴权启动，一切都完毕之后，在mongo router中设置账号密码，再所有的机器带鉴权全部重启一遍
2. 一开始手贱非要先设置账号密码，那就各个分片的副本集、router、config的账号密码保持一致即可


# 9 注意点

1. shard key至少是其中一个索引的第一列。

如果表/集合本身是空的，直接：
```
sh.shardCollection("sharding_db_test.jeason_test",{"user_id":1})
```
即可，会自动给user_id创建索引。

如果表/集合本身已经存在数据了，且没有以分片属性开始的索引，则先创建索引，再创建分片：

```
<!-- 创建索引 -->
use sharding_db_test
db.jeason_test.ensureIndex({"user_id":1})

<!-- 创建分片 -->
sh.shardCollection("sharding_db_test.jeason_test",{"user_id":1})
```

对于本身有已经有数据，且缺少以分片列开始的索引，则会报错：please create an index that starts with the shard key before sharding 

2. 帐号权限问题
只有root权限的帐号才能开启某个db的分片，从而有权限开启某个集合的分片并设置分片使用的key
