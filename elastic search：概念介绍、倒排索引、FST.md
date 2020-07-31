# 1 背景
参考文档：

安装和配置     https://www.cnblogs.com/jhxxb/p/11190756.html

简单的增删查改    https://www.jianshu.com/p/7d687c9dba4f

常用命令   https://www.jianshu.com/p/c6708ea88710

ES基本概念介绍   https://blog.51cto.com/mageedu/1714522?utm_source=tuicool&utm_medium=referral

ES数据类型    https://www.cnblogs.com/betterwgo/p/11571275.html

ES索引原理分析（倒排索引、Finite State Transducers）  https://blog.csdn.net/qq_38262266/article/details/90311086

Elasticsearch入门博客（一整套） https://www.cnblogs.com/ginb/p/6637236.html

mapping

# 2 安装





## 2.x 配置本机之外可访问
默认配置是/etc/elasticsearch/elasticsearch.yml，其中的network.host默认是127.0.0.1，也就是，只监听127.0.0.1这个IP上过来的请求，也就是默认配置情况下，只能本地访问。

为了配置为本机之外的IP访问，需要修改一下配置：

1. network.host修改为 0.0.0.0  , 监听本机的所有网卡和IP
2. 取消注释node.name: node-161
3. 取消注释cluster.initial_master_nodes: ["node-161"]，并将初始主节点设置为自己

最后，systemctl restart elasticsearch即可，再用局域网的另一台的机器即可访问
