# 1 背景
参考文档：

ES官方文档   https://www.elastic.co/guide/en/elasticsearch/reference/7.9/modules-cluster.html#shard-allocation-awareness

通过ES的副本集和节点标签，实现一个地理位置上节点故障后的分片转移功能，核心配置如下：

```yml
cluster.routing.allocation.awareness.attributes: zone
cluster.routing.allocation.awareness.force.zone.values: zone1,zone2,zone3

# 根据情况，取zone1、zone2、zone3
node.attr.zone: zone1
```

# 2 详细介绍
