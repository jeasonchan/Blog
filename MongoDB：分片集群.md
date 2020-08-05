# 1 背景

参考文档：

MongoDB Sharded cluster架构原理  https://developer.aliyun.com/article/32434


# 2 为什么要使用Sharded cluster
MongoDB目前3大核心优势：
* 灵活模式，通过json文档来实现灵活模式
* 高可用性，通过复制集来保证高可用
* 可扩展性，通过Sharded cluster来保证可扩展性

当MongoDB复制集遇到下面的业务场景时，你就需要考虑使用Sharded cluster：

* 存储容量需求超出单机磁盘容量
* 活跃的数据集超出单机内存容量，导致很多请求都要从磁盘读取数据，影响性能
* 写IOPS超出单个MongoDB节点的写服务能力

![aaa](https://yqfile.alicdn.com/4eed80bbaa26f79ad12a74e9113c744d5e77ccf8.jpeg)
