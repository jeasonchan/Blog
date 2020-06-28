@[TOC]
# 1 原理介绍
# 2 平台搭建
## 2.1 Windows平台
## 2.2linux平台
# 3 java代码实现
## 3.1 生产者
## 3.2 消费者
多个消费者实例采用相同group id去监听同一个主题，可能会因为负载均衡问题，丢失部分该主题下的消息，有时候甚至会完全收不到消息。
每一个生产者放入kafka的json串都是消费者取到的ConsumerRecords中的ConsumerRecord中的value()取到的string，即，Consumer<K,V>中的V类型即为生产者放到kafka中的类型
