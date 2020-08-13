# 1 背景
参考文档：

安装和配置     https://www.cnblogs.com/jhxxb/p/11190756.html

简单的增删查改    https://www.jianshu.com/p/7d687c9dba4f

常用命令   https://www.jianshu.com/p/c6708ea88710

ES基本概念介绍   https://blog.51cto.com/mageedu/1714522?utm_source=tuicool&utm_medium=referral

ES数据类型    https://www.cnblogs.com/betterwgo/p/11571275.html

ES索引原理分析（倒排索引、Finite State Transducers）  https://blog.csdn.net/qq_38262266/article/details/90311086

Elasticsearch入门博客（一整套） https://www.cnblogs.com/ginb/p/6637236.html

ElasticSearch——路由(_routing)机制     https://www.cnblogs.com/caoweixiong/p/12029789.html

# 2 单机部署

直接在yum install安装即可，安装完会自动在/usr/lib/systemd/system/中添加守护进程文件，自动添加配置文件/etc/elasticsearch/elasticsearch.yml。

## 2.x 配置本机之外可访问
默认配置是/etc/elasticsearch/elasticsearch.yml，其中的network.host默认是127.0.0.1，也就是，只监听127.0.0.1这个IP上过来的请求，也就是默认配置情况下，只能本地访问。

为了配置为本机之外的IP访问，需要修改一下配置：

1. network.host修改为 0.0.0.0  , 监听本机的所有网卡和IP
2. 取消注释node.name: node-161,将节点名字设为node-161
3. 取消注释cluster.initial_master_nodes: ["node-161"]，并将初始主节点设置为自己

修改后的部分配置如下：
```yml
node.name: node-161
network.host: 0.0.0.0
cluster.initial_master_nodes: ["node-161"]
```

最后，systemctl restart elasticsearch即可，再用局域网的另一台的机器即可访问

# 3 集群部署

## 3.1 配置文件
相比于单机部署，集群部只多了几个的配置参数，并且不同集群节点的配置，仅仅只有一个的node.name是互不相同的，其余的配置参数都是相同的。

完整配置如下：

```yml
cluster.name: escluster
node.name: es1
node.master: true
node.data: true
path.data: /data/elasticsearch/data
path.logs: /data/elasticsearch/logs
bootstrap.memory_lock: true
bootstrap.system_call_filter: false
http.port: 9200
network.host: 0.0.0.0
discovery.zen.minimum_master_nodes: 2
discovery.zen.ping_timeout: 300s
discovery.zen.ping.unicast.hosts: ["xx.xx.xx.8:9300","xx.xx.xx.6:9300","xx.xx.xx.9:9300"]
```

三个节点的配置文件，唯一不同的就是node.name配置项：

"xx.xx.xx.8:9300"  node.name: es1

"xx.xx.xx.6:9300"  node.name: es2

"xx.xx.xx.9:9300"  node.name: es3


实际成功集群所用的配置文件，配置参数有出入，要详细确认一下：

```yml
cluster.name: wic
node.name: node-97
path.data: /var/lib/elasticsearch
path.logs: /var/log/elasticsearch
bootstrap.memory_lock: true
network.host: 0.0.0.0
network.publish_host: 10.40.65.97
http.port: 9200
transport.port: 9300
discovery.seed_hosts:
  - node-97:9300
  - node-98:9300
  - node-99:9300
cluster.initial_master_nodes: ["node-99","node-97","node-98"]
```


### 3.1.1 配置参数详解
（1）cluster.name
集群名字，三个节点的配置文件中，该参数的值必须一致

（2）node.name
当前节点的名字，三台ES节点字都必须不一样

（3）discovery.zen.minimum_master_nodes:2
表示集群最少的master数，如果集群的最少master数据少于指定的数，将无法启动，官方推荐minimum_master_nodes设置为  集群节点数/2+1，我这里三台ES节点服务器，配置最少需要 3/2+1=2 台master，整个集群才可正常运行，

（4）node.master
该节点是否有资格选举为master，**如果上面设了minimum_master_nodes=2，也就是最少两个master节点**，则集群中必须有两台es服务器的配置为node.master: true的配置。配置了2个节点的话，如果主服务器宕机，整个集群会不可用，所以三台服务器，需要配置3个node.masdter为true,这样三个master，宕了一个主节点的话，他又会选举新的master，还有两个节点可以用，只要配了node master为true的ES服务器数正在运行的数量不少于master_node的配置数，则整个集群继续可用，我这里则配置三台es node.master都为true，也就是三个master，master服务器主要管理集群状态，负责元数据处理，比如索引增加删除分片分配等，数据存储和查询都不会走主节点，压力较小，jvm内存可分配较低一点

（5）node.data
存储索引数据，三台都设为true即可

（6）bootstrap.memory_lock: true
锁住物理内存，不使用swap内存，有swap内存的可以开启此项

（7）discovery.zen.ping_timeout: 300s
自动发现拼其他节点超时时间

（8）discovery.zen.ping.unicast.hosts: ["xx.xx.xx.8:9300","xx.xx.xx.6:9300","xx.xx.xx.9:9300"]
设置集群的初始节点列表，**集群互通**端口为9300，和http.port这个rest接口使用的9200端口不是一回事

## 3.2 JVM参数调优


## 3.3 调整内核参数


## 3.4 设置守护进程


## 3.5 设置目录权限
根据守护进程中，启动ES所使用的用户，给相关目录给予使用的用户相关权限


## 3.6 安装离线插件
1. 下载ES的插件包，一般是plugin-name.zip，上传到Linux某目录中，比如：/path/to/plugin-name.zip

2. 刷一下权限，chmod 777 plugin-name.zip，以防万一

3. cd到ES的home目录（默认是/usr/share/elasticsearch）

4. 使用ES的插件执行文件安装第一步下载的zip包
```bash
cd /usr/share/elasticsearch

#注意！！！！！！从本地安装必须要使用file://开头，表明是从本地安装
bin/elasticsearch-plugin install  file:///path/to/plugin-name.zip
```
5. bin/elasticsearch-plugin list  查看已安装的插件

最后，看一下整个命令行：
```bash
chmod 777 /root/jeason/opendistro_sql-1.9.0.0.zip ;/usr/share/elasticsearch/bin/elasticsearch-plugin install  file:///root/jeason/opendistro_sql-1.9.0.0.zip;/usr/share/elasticsearch/bin/elasticsearch-plugin list

#以下是输出
future versions of Elasticsearch will require Java 11; your Java version from [/usr/java/jdk1.8.0_121/jre] does not meet this requirement
-> Installing file:///root/jeason/opendistro_sql-1.9.0.0.zip
-> Downloading file:///root/jeason/opendistro_sql-1.9.0.0.zip
[=================================================] 100%  
-> Installed opendistro_sql
future versions of Elasticsearch will require Java 11; your Java version from [/usr/java/jdk1.8.0_121/jre] does not meet this requirement
opendistro_sql
```

**插件安装完必须重启才能生效**

## 3.7 监控API

```
GET  /_cat
 
/_cat/health
/_cat/nodes
/_cat/master
/_cat/indices
/_cat/allocation 
/_cat/shards 
/_cat/shards/{index}
/_cat/thread_pool
/_cat/segments 
/_cat/segments/{index}
```
比如：http://{{IP}}:9200/_cat/nodes?pretty=true&v=true

# 9 补充

## 9.1 查询索引状态
GET   http://{{IP}}:9200/_cat/indices?v=true&pretty=true

```
health status index                          uuid                   pri rep docs.count docs.deleted store.size pri.store.size
green  open   .kibana_task_manager_2         oz92mEhTRPu95-K7rmnspA   1   1          6            6     94.6kb         55.3kb
green  open   .apm-custom-link               usD-G5vhQpCHH_Zn3EMa8g   6   1          0            0      2.4kb          1.2kb
green  open   .kibana_task_manager_1         jemsr5oAQKCZ2YMH891S4g   1   1          2            0     25.3kb         12.6kb
green  open   .apm-agent-configuration       JOtQTvxvRrq6oAbfXmhN5g   1   1          0            0       416b           208b
green  open   jeason_test_index              UO2wgOr2QRW06fWOnfWShA   6   0          0            0      1.2kb          1.2kb
green  open   .kibana_2                      BUCMsu21Qk26QFlnOU42Sg   1   1         43            0    137.5kb           72kb
green  open   .kibana-event-log-7.8.1-000001 uPRJrkrdRh6G_q7ana4R_A   6   1          3            0     33.1kb         16.5kb
green  open   .kibana_1                      463NBSDvRyCTBCcxG3_tww   1   1          8            0     65.8kb         32.9kb

```
对索引的状态进行解释：

有两个重要的参数：

primary：就是number_of_shards数量，单节点也可以定义多个shard

replica：就是number_of_replicas，副本集，单机的ES只能是0，只有ES集群才能的设置多副本集，副本集是对所有primary的备份


## 9.2 创建索引
1. PUT http://{{IP}}:9200/索引名 可以不带body，body样例如下：
```
{
    "settings": {
        "index": {
            "number_of_shards": 2,
            "number_of_replicas": 0
        }
    }
}
```
不带body时，使用默认的索引模板，也就是：1P（主） 1R（副本），如果ES只有一个节点，那R是未赋值的状态

索引名规定不能包含大写，否则接口会会返回异常

## 9.3 创建索引模板
PUT  http://{{IP}}:9200/_template/template_name

```
{
    "index_patterns": [
        "*"
    ],
    "order": 1,
    "settings": {
        "number_of_shards": 6,
        "number_of_replicas": 0
    }
}
```
body里面提到的设置会直接使用，没提到的设置会使用默认的值。


## 9.4 路由router介绍
ES中路由计算时，使用了哈希+取模计算落在哪个shard，所以，无论指定什么routing值去搜索，必然会且只在1个shard上进行搜索

## 9.5 查看集群状态
GET http://{{IP}}:9200/_cat/nodes?pretty=true&v=true

```
ip          heap.percent ram.percent cpu load_1m load_5m load_15m node.role master name
10.40.65.99           68          86   0    0.84    0.36     0.17 dilmrt    -      node-99
10.40.65.97           66          55   0    0.00    0.03     0.05 dilmrt    *      node-97
10.40.65.98           23          70   0    0.08    0.10     0.08 dilmrt    -      node-98
```

## 9.6 增加文档
POST http://{{IP}}:9200/索引名/_doc

文档对象，json格式
```
{
    "name":"jeason"
}
```

## 9.7 查看某索引的文档数量
GET http://{{IP}}:9200/_cat/count/索引名

```
1597288862 03:21:02 0
```


## 9.8 

