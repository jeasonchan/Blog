# 1 背景
数据存储在MongoDB中，需要用到全文索引，于是想到和ES组合使用，但是，问题来了，考虑分布式且高可用的场景下，如何将MongoDB数据同步ES？

目前了解到方式主要有三大类：

1、使用logStash，将MongoDB数据源作输入，ES作为输出

2、使用MongoDB-connector，已经不维护了，放弃……但是，项目代码可以看一看https://github.com/yougov/mongo-connector

3、Monstache

同时，从MongoDB取数据还分为两种方式：

1、oplog

2、changeStream


# 2 各方案简介
## 2.1 logStash
参考文档：

stack OverFlow上的回答   https://stackoverflow.com/a/56631136


You may sync MongoDB and Elasticsearch with Logstash; syncing is, in fact, one of the major applications of Logstash. After installing Logstash, all that you need to do is specify a pipeline for your use case: one or more input sources (MongoDB in your case) and one or more output sinks (Elasticsearch in your case), put as a config file (example follows) inside Logstash's pipeline directory; Logstash takes care of the rest.

Logstash officially provides plugins for a lot of commonly used data sources and sinks; those plugins let you read data from and write data to various sources with just a few configuration. You just need to find the right plugin, install it, and configure it for your scenario. Logstash has an official output plugin for Elasticsearch and its configuration is pretty intuitive. Logstash, however, doesn't provide any input plugin for MongoDB. You need to find a third party one; this one seems to be pretty promising.

In the end, your pipeline may look something like the following:

```
input {
  mongodb {
    uri => 'mongodb://10.0.0.30/my-logs?ssl=true'
    placeholder_db_dir => '/opt/logstash-mongodb/'
    placeholder_db_name => 'logstash_sqlite.db'
    collection => 'events_'
    batch_size => 5000
  }
}
output {
  stdout {
    codec => rubydebug #outputs the same thing as elasticsearch in stdout to facilitate debugging
  }
  elasticsearch {
    hosts => "localhost:9200"
    index => "target_index"
    document_type => "document_type"
    document_id => "%{id}"
  }
}
```

但是，这个input插件存在一定的问题：

Note on that input plugin: "This was designed for parsing logs that were written into mongodb. This means that it may not re-parse db entries that were changed and already parsed." So, only useful in a very limited number of use cases.

logStash不能拿来直接用，还要改造，暂时再看看别的方案。

## 2.2 MonStash

官方网站：https://rwynn.github.io/monstache-site/


特性：
Support for MongoDB change streams and aggregation pipelines

用法：
1、默认使用oplog作为事件源

Monstache uses the MongoDB oplog as an event source. You will need to ensure that MongoDB is configured to produce an oplog by deploying a replica set. 

**就算是使用change stream，也需要配置一个复制集，毕竟change stream基于oplog**


2、在链接串的字符中配置恰当用户名，以保证monstash读数据库、创建索引

The user in the connection string will need to be able read the local database (to read from the oplog) and any user databases that you wish to synch data from.


3、使用change stream作为数据源时，需要自己实现一部分东西

When using change streams you will need to implement the changes in the documentation for access control（https://docs.mongodb.com/manual/changeStreams/#access-control）. Monstache defaults to opening the change stream against the entire deployment.

4、不做任何配置的情况下的，monstash会从localhost的MongoDB的默认端口读取oplog，并放到localhost的ES的默认端口中。所以，ES和MongoDB部署在不同主机上时，需要修改配置

Without any explicit configuration monstache will connect to Elasticsearch and MongoDB on localhost on the default ports and begin tailing the MongoDB oplog. Any changes to MongoDB while Monstache is running will be reflected in Elasticsearch.

5、查看monstash在ES中创建的索引，使用命令：curl localhost:9200/_cat/indices?v

To see the indexes created by Monstache you may want to issue the following command which will show the indices in Elasticsearch. By default, the index names will match the db.collection name in MongoDB.

6、以特定的配置启动monstash：monstache -f /path/to/config.toml

Monstache uses the TOML（https://github.com/toml-lang/toml） format for its configuration. You can run monstache with an explicit configuration by passing the -f flag.

配置文件样例：

```toml
# connection settings

# connect to MongoDB using the following URL
mongo-url = "mongodb://someuser:password@localhost:40001"
# connect to the Elasticsearch REST API at the following node URLs
elasticsearch-urls = ["https://es1:9200", "https://es2:9200"]

# frequently required settings

# if you need to seed an index from a collection and not just listen and sync changes events
# you can copy entire collections or views from MongoDB to Elasticsearch
direct-read-namespaces = ["mydb.mycollection", "db.collection", "test.test", "db2.myview"]

# if you want to use MongoDB change streams instead of legacy oplog tailing use change-stream-namespaces
# change streams require at least MongoDB API 3.6+
# if you have MongoDB 4+ you can listen for changes to an entire database or entire deployment
# in this case you usually don't need regexes in your config to filter collections unless you target the deployment.
# to listen to an entire db use only the database name.  For a deployment use an empty string.
change-stream-namespaces = ["mydb.mycollection", "db.collection", "test.test"]

# additional settings

# if you don't want to listen for changes to all collections in MongoDB but only a few
# e.g. only listen for inserts, updates, deletes, and drops from mydb.mycollection
# this setting does not initiate a copy, it is only a filter on the change event listener
namespace-regex = '^mydb\.mycollection$'
# compress requests to Elasticsearch
gzip = true
# generate indexing statistics
stats = true
# index statistics into Elasticsearch
index-stats = true
# use the following PEM file for connections to MongoDB
mongo-pem-file = "/path/to/mongoCert.pem"
# disable PEM validation
mongo-validate-pem-file = false
# use the following user name for Elasticsearch basic auth
elasticsearch-user = "someuser"
# use the following password for Elasticsearch basic auth
elasticsearch-password = "somepassword"
# use 4 go routines concurrently pushing documents to Elasticsearch
elasticsearch-max-conns = 4 
# use the following PEM file to connections to Elasticsearch
elasticsearch-pem-file = "/path/to/elasticCert.pem"
# validate connections to Elasticsearch
elastic-validate-pem-file = true
# propogate dropped collections in MongoDB as index deletes in Elasticsearch
dropped-collections = true
# propogate dropped databases in MongoDB as index deletes in Elasticsearch
dropped-databases = true
# do not start processing at the beginning of the MongoDB oplog
# if you set the replay to true you may see version conflict messages
# in the log if you had synced previously. This just means that you are replaying old docs which are already
# in Elasticsearch with a newer version. Elasticsearch is preventing the old docs from overwriting new ones.
replay = false
# resume processing from a timestamp saved in a previous run
resume = true
# do not validate that progress timestamps have been saved
resume-write-unsafe = false
# override the name under which resume state is saved
resume-name = "default"
# use a custom resume strategy (tokens) instead of the default strategy (timestamps)
# tokens work with MongoDB API 3.6+ while timestamps work only with MongoDB API 4.0+
resume-strategy = 1
# exclude documents whose namespace matches the following pattern
namespace-exclude-regex = '^mydb\.ignorecollection$'
# turn on indexing of GridFS file content
index-files = true
# turn on search result highlighting of GridFS content
file-highlighting = true
# index GridFS files inserted into the following collections
file-namespaces = ["users.fs.files"]
# print detailed information including request traces
verbose = true
# enable clustering mode
cluster-name = 'apollo'
# do not exit after full-sync, rather continue tailing the oplog
exit-after-direct-reads = false

```

### 2.2.1 配置参数详解
各个配置参数的详细解释，见官方文档：https://rwynn.github.io/monstache-site/config/#mongo-config-url


**change-stream-namespaces**

如果配置此参数，则使用change stream（MongoDB > 3.6 version）作为数据源，change stream有个天然优势，整个分片集群不需要分别监听，oplog需要单独分别监听各个分片的节点。使用change stream后，自动禁用监听oplog。


**cluster-name**

设置monstash集群模式：When cluster-name is given monstache will enter a high availablity mode. Processes with cluster name set to the same value will coordinate. Only one of the processes in a cluster will sync changes. The other processes will be in a paused state. If the process which is syncing changes goes down for some reason one of the processes in paused state will take control and start syncing. See the section high availability for more information.


**elasticsearch-urls**

elasticsearch-urls = ["https://es1:9200", "https://es2:9200"]，因为ES必定是集群的，这个参数用这种方式没什么问题。


**mongo-url**

单节点可以直接使用# connect to MongoDB using the following URL
mongo-url = "mongodb://someuser:password@localhost:40001"。连接串语法遵循（https://docs.mongodb.com/v3.0/reference/connection-string/#standard-connection-string-format）。

对于MongoDB的分片集群部署的方式，For sharded clusters this URL should point to the mongos router server and the **mongo-config-url** option must be set to point to the config server.


**mongo-config-url**
This config must only be set for sharded MongoDB clusters. Has the same syntax as mongo-url. This URL must point to the MongoDB config server.

Setting the mongo-config-url is not necessary if you are using **change-stream-namespaces**。可见，该参数主要用于去各个分片节点读取oplog

### 2.2.2 高级设置/功能

#### 2.2.2.1 多worker模式
Workers   https://rwynn.github.io/monstache-site/advanced/#workers

配置项：
```
workers = ["Tom", "Dick", "Harry"]
worker="Tom"
```
在单机上启动多个monstache进程，每个进程都是一个worker节点，每个worker节点只负责同步mongo中特定ID的文档/记录，多个worker配合工作才能同步所有的文档。

#### 2.2.2.2 Routing
Routing功能介绍   https://rwynn.github.io/monstache-site/advanced/#routing

monstache默认情况下会使用 db_name.collection_name 作为 ES 中的索引名，并且，这个是可以自定义映射的。

MongoDB有分片集群的部署模式，分片的单位是collection，一个collection中的文档对象会根据对象的某个 key/属性 按照某种规则放到某个shard（分片）中，没有开启分片的collection会全都放到主分片中。

默认情况下，ES本身会使用插入的文档对象的文档id作为ES自身分片路由的key，但是，由于文档id本身是无规律的，文档会无规律的分到各个shard里面，不带路由key查询时会在所有分片广播搜索，效率低。

可见，MongoDB分片使用的key和ES路由所用的key有种天然的对应关系。

monstache支持在配置文件中手动指定某个集合向ES中映射使用的key，指定blog.comments集合向ES插入时，使用文档对象的post_id作为路由的属性：

```
[[script]]
namespace = "blog.comments"
routing = true
script = """
module.exports = function(doc) {
    doc._meta_monstache = { routing: doc.post_id };
    return doc;
}
"""
```

假设集合blog.comments中，所有文档都有属性post_id，并且是离散值：1~10，向ES中插入数据时，就会将monstache会根据配置文件中的配置，将文档对象的post_id取出来，插入时会用到该值：

```
<!-- ES会根据routing=1将该文档放到相应的shard中 -->
PUT blog.comments/_doc/c?routing=1&refresh   {"post_id":1,xxxx:xxxxxx}

<!-- ES会根据routing=3将该文档放到相应的shard中 -->
PUT blog.comments/_doc/c?routing=3&refresh   {"post_id":3,xxxx:xxxxxx}

```

之后，在ES中向搜索blog.comments索引中的特定路由(比如，routing=3)，可以着这样：

```
$ curl -H "Content-Type:application/json" -XGET 'http://localhost:9200/blog.comments/_search?routing=3' -d '
{
   "query":{
      "match_all":{}
   }
}'
```

monstache能否自动发现集合所使用的shard key，并且，在向ES插入数据时自动使用该key？因为，之后可能会动态增加集合，肯定不能每次都暂停monstache手动增加routing映射的。

经过实践，无法自动发现MongoDB的shard key，为了能使新增的集合也能根据shard key在ES中根据路由放进shard，需要使用上面的脚本添加_meta_monstache属性，同时，所有document都应该设置一个一致的shard key属性，比如就叫：shard_key，脚本里面就可以统一方便写成：

```
module.exports = function(doc) {
    doc._meta_monstache = { routing: doc.shard_key };
    return doc;
}
```

#### 2.2.2.3 ES数据插入机制

MongoDB数据PUT到ES中时，不会每次都refresh，而是首次搜索时才会refresh


### 2.2.3 验证单机数据同步、类型转换
所有实例都在同一台机器上：

MongoDB      0.0.0.0:27017
MongoDB(复制集主节点)  0.0.0.0:27018 用户名admin 密码admin
elasticsearch 0.0.0.0:9200

#### 2.2.3.1 部署Monstache
部署过程比较简单，monstache本身只有1个可执行文件，无需将可执行文件软/硬连接到/usr/binmonstache 即可。部署分为两步：

1、编辑配置文件；

2、使用monstache -f ./monstache.conf运行，这种运行方式会讲输出绑定到当前窗口，后台执行需要配合使用nohup。

单机部署的配置文件如下：
```
[root@searchcode-02 monstache]# cat monstache_conf.toml

# MongoDB和ES的配置按实际设置即可
mongo-url = "mongodb://jeason:jeason@localhost:27018,localhost:27017"
elasticsearch-urls = ["http://localhost:9200"]
elasticsearch-max-seconds = 10
elasticsearch-max-conns = 12

# 不明确，就监控整个mongo中所有集合的change stream
change-stream-namespaces = [""]

# 打印详细的日志
verbose = true

# 不明确，就同步整个mongo中的数据
direct-read-namespaces = [""]

gzip = true

# 同步整mongo后，不退出（false）monstache，
# 继续根据oplog或者change stream增量同步数据
exit-after-direct-reads = false
```

一开始启动的时候可能会报错，比如change stream的鉴权失败什么的。建议在mongo中新建一个用户，赋予root级别的权限。比如：

```
<副本集ID>:PRIMARY> use admin
switched to db admin

<副本集ID>:PRIMARY> show users
{
        "_id" : "admin.admin",
        "user" : "admin",
        "db" : "admin",
        "roles" : [
                {
                        "role" : "dbAdminAnyDatabase",
                        "db" : "admin"
                },
                {
                        "role" : "userAdminAnyDatabase",
                        "db" : "admin"
                }
        ],
        "mechanisms" : [
                "SCRAM-SHA-1",
                "SCRAM-SHA-256"
        ]
}
{
        "_id" : "admin.jeason",
        "user" : "jeason",
        "db" : "admin",
        "roles" : [
                {
                        "role" : "root",
                        "db" : "admin"
                }
        ],
        "mechanisms" : [
                "SCRAM-SHA-1",
                "SCRAM-SHA-256"
        ]
}

```

#### 2.2.3.2 数据同步验证
在mongo中增加一些数据：

```
mongo

use admin

db.auth("jeason","jeason")

db.insert({"name","hahaha","age":666})
db.insert({"name","hahaha","age":777})
```

在ES中查看所有的索引库和其下的文档数量：http://{{IP}}:9200/_cat/indices?v
```
health status index                      uuid                   pri rep docs.count docs.deleted store.size pri.store.size
green  open   search_repo_1              iGTYGngkRkWkZi_t_zp4lQ   1   0          0            0       208b           208b
yellow open   mydb.mycollection          RPOUWA1GRw2ncKSPxevkBQ   1   1          8            0     21.5kb         21.5kb
yellow open   monstache.stats.2020-07-30 O3lyrW1PSyiOyALItJMW3A   1   1         80            0       72kb           72kb
```


查看mydb.mycollection索引库下的所有文档：http://{{IP}}:9200/mydb.mycollection/_search?pretty=true
```
{
  "took": 3,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 8,
      "relation": "eq"
    },
    "max_score": 1.0,
    "hits": [
      {
        "_index": "mydb.mycollection",
        "_type": "_doc",
        "_id": "5f22f639dd980b605f8fdc9d",
        "_score": 1.0,
        "_source": {
          "age": 300,
          "name": "jea2521sw11on"
        }
      },
      {
        "_index": "mydb.mycollection",
        "_type": "_doc",
        "_id": "5f22f8f42688de9e67e4f49f",
        "_score": 1.0,
        "_source": {
          "list": [
            "123",
            "456",
            "789"
          ]
        }
      }]
  }
}
```

查看ES中mydb.mycollection索引库中的映射关系（**？？？？？？**）：http://{{IP}}:9200/mydb.mycollection/_mapping?pretty=true

```
{
  "mydb.mycollection": {
    "mappings": {
      "properties": {
        "age": {
          "type": "long"
        },
        "list": {
          "type": "text",
          "fields": {
            "keyword": {
              "type": "keyword",
              "ignore_above": 256
            }
          }
        },
        "name": {
          "type": "text",
          "fields": {
            "keyword": {
              "type": "keyword",
              "ignore_above": 256
            }
          }
        }
      }
    }
  }
}
```

同样的，在mongo中删除、修改数据，ES中的源文档也相应同步变化。

**结论：MongoDB中增删改，都会同步变更到ES中源文档中。**

### 2.2.4 验证双机Monstash集群
Monstache作为中间件，需充分考虑高可用的情况，参考的文档：

https://rwynn.github.io/monstache-site/config/#cluster-name

https://rwynn.github.io/monstache-site/advanced/#high-availability

#### 2.2.4.1 高可用实现原理
1. 所有的monstache形成集群，确保MongoDB数据一直同步到elasticsearch
2. 在MongoDB中新建monstache（库），然后建立cluster(集合/表)，在其中记录当前活动的某个节点名，也就是，**如果这个集合中没有记录，则表明monstache集群全挂了或者没有开启高可用/集群模式**
3. 只有monstache.cluster中的monstache节点会执行数据同步任务，其余的节点完全不工作
4. monstache.cluster表中记录当前工作monstache节点的文档/记录有过期时间，工作节点会定期更新整个节点已表明自己在线，常见不更新文档则意味着自己挂了，MongoDB到期删除该条文档/记录，其他monstache检测到当前无工作节点了，开始选举出工作节点，工作节点重新向 monstache.cluster 中写入带有过期时间的记录。

总的来说，monstache的高可用的检测原理，有点类似于基于zookeeper临时节点的服务在线检测，临时节点不在了则说明服务挂了。

#### 2.2.4.2 新增的配置文件项
1. 设置各个monstache节点的cluster-name为同一个值，开启集群模式
2. 显式设置resume=true，尽管开启集群模式后，默认变为true；开启后，工作过节点会将自己的集群ID、自己最后一次成功过同步的时间戳写进MongoDB的monstache.monstache中，如：

| _id | ts |
| -- | --- |
| apollo | Timestamp{value=6855305087644860418, seconds=1596125096, inc=2} |

3. 设置resume-name。一般情况下，开启集群/高可用模式的情况下，resume-name会自动代替cluster-name，在开启了多worker模式的情况下，worker的id还会进一步替代resume-name

修改之后的配置文件如下：

```
mongo-url = "mongodb://jeason:jeason@localhost:27018,localhost:27017"

elasticsearch-urls = ["http://localhost:9200"]

elasticsearch-max-seconds = 10

elasticsearch-max-conns = 12

change-stream-namespaces = [""]

verbose = true

direct-read-namespaces = [""]

gzip = true

exit-after-direct-reads = false

cluster-name = "cluster-name"

resume = true

resume-name = "resume-name"
```
#### 2.2.4.3 启动高可用集群
在需要集群的机器上，直接使用 monstache -f the_same_conf.toml 文件启动多个的monstache进程即可。

第一个启动的会有如下日志打印，且正在Watching changes on the deployment：
```
[root@searchcode-02 monstache]# monstache -f ./monstache_conf.toml
INFO 2020/07/31 17:07:15 Started monstache version 6.6.0
INFO 2020/07/31 17:07:15 Go version go1.14.1
INFO 2020/07/31 17:07:15 MongoDB go driver v1.3.4
INFO 2020/07/31 17:07:15 Elasticsearch go driver 7.0.17
INFO 2020/07/31 17:07:15 Successfully connected to MongoDB version 4.0.5
INFO 2020/07/31 17:07:15 Successfully connected to Elasticsearch version 7.8.0
INFO 2020/07/31 17:07:15 Sending systemd READY=1
WARN 2020/07/31 17:07:15 Systemd notification not supported (i.e. NOTIFY_SOCKET is unset)
INFO 2020/07/31 17:07:15 Joined cluster cluster-name
INFO 2020/07/31 17:07:15 Pausing work for cluster cluster-name
INFO 2020/07/31 17:08:25 Resuming work for cluster cluster-name
INFO 2020/07/31 17:08:25 Dynamic direct read candidates: [myDB.myCollection]
INFO 2020/07/31 17:08:25 Listening for events
INFO 2020/07/31 17:08:25 Watching changes on the deployment
INFO 2020/07/31 17:08:25 Resuming from timestamp {T:1596186395 I:3}
INFO 2020/07/31 17:08:25 Direct reads completed
TRACE 2020/07/31 17:08:35 POST /_bulk HTTP/1.1
Host: localhost:9200
User-Agent: elastic/7.0.17 (linux-amd64)
Content-Length: 145
Accept: application/json
Content-Type: application/x-ndjson
Accept-Encoding: gzip
```

之后启动的会有如下打印日志，表明已加入集群，但是Pausing work for cluster cluster-name：
```
[root@searchcode-02 monstache]# monstache -f ./monstache_conf.toml
INFO 2020/07/31 17:08:36 Started monstache version 6.6.0
INFO 2020/07/31 17:08:36 Go version go1.14.1
INFO 2020/07/31 17:08:36 MongoDB go driver v1.3.4
INFO 2020/07/31 17:08:36 Elasticsearch go driver 7.0.17
INFO 2020/07/31 17:08:36 Successfully connected to MongoDB version 4.0.5
INFO 2020/07/31 17:08:36 Successfully connected to Elasticsearch version 7.8.0
INFO 2020/07/31 17:08:36 Sending systemd READY=1
WARN 2020/07/31 17:08:36 Systemd notification not supported (i.e. NOTIFY_SOCKET is unset)
INFO 2020/07/31 17:08:36 Joined cluster cluster-name
INFO 2020/07/31 17:08:36 Pausing work for cluster cluster-name
```

就像文档说的那样，只有一个会工作进行数据同步，其余的都会暂停。

尝试kill掉正在同步的monstache节点后，其余节点的状态自动发生了变化，开始成为工作节点，同一台机器中，切点切换时间大概为1秒。

结论：出版判断monstache具备高可用性

### 2.2.5 验证Monstash双机集群+MongoDB分片
插入数据后，数据同步到ES延迟较为严重：

1. 插入数据后，change stream产生较慢，原因：https://docs.mongodb.com/manual/administration/change-streams-production-recommendations/#sharded-clusters

初步怀疑是分片集群模式下，change stream本身产生较慢，果然也有相应的解释的：https://docs.mongodb.com/manual/administration/change-streams-production-recommendations/#sharded-clusters

于是写代码验证一下，对change stream的延迟进行量化，分别用java写一个change stream监控服务器和一个的MongoDB的文档插入服务器：

#### 2.2.5.1 change stream的监控服务器
MongoDB驱动：
```
<dependency>
    <groupId>org.mongodb</groupId>
    <artifactId>mongodb-driver</artifactId>
    <version>3.8.2</version>
</dependency>
```

```java
package default_package.mongo_test;

import com.mongodb.client.*;
import com.mongodb.client.model.changestream.ChangeStreamDocument;
import org.bson.Document;

import java.net.HttpURLConnection;
import java.util.Date;

public class WatchChangeStream {
    public static void main(String[] args) {
        MongoClient mongoClient = null;
        HttpURLConnection connection = null;
        try {


            mongoClient = MongoClients.create("mongodb://xx.xx.xx.xx:31000");
            MongoDatabase myDB = mongoClient.getDatabase("sharding_db_test");
            MongoCollection<Document> myCollection = myDB.getCollection("jeason_test3");
            String indexName = "sharding_db_test.jeason_test3";


            MongoCursor<ChangeStreamDocument<Document>> cursor = myCollection.watch().iterator();

            while (true) {

                while (cursor.hasNext()) {
                    System.out.println(new Date() + "：");
                    System.out.println(cursor.next());
                }

            }


        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

#### 2.2.5.2 MongoDB数据插入
同样依赖上文的MongoDB驱动。

```java
package default_package.mongo_test;

import com.mongodb.client.MongoClient;
import com.mongodb.client.MongoClients;
import com.mongodb.client.MongoCollection;
import com.mongodb.client.MongoDatabase;
import org.bson.Document;

import java.util.Date;
import java.util.concurrent.CountDownLatch;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.TimeUnit;


public class InsertAndQueryES {
    private static String IP = "";


    public static Integer get(String indexName) {
        String url = "http://xx.xx.xx.xx:9200/_cat/count/" + indexName;
        Object res = RestTemplateUtil.getUrl(url, null);
        return Integer.parseInt(String.valueOf(res).split(" ")[2].trim());
    }


    public static void main(String[] args) {
        MongoClient mongoClient = null;
        try {


            mongoClient = MongoClients.create("mongodb://xx.xx.xx.xx:31000");
            MongoDatabase myDB = mongoClient.getDatabase("sharding_db_test");
            MongoCollection<Document> myCollection = myDB.getCollection("jeason_test3");
            String indexName = "sharding_db_test.jeason_test3";

            int cycle = 10;
            int threadNumber = 4;
            int numberForEachThread = 100;

            if (cycle * threadNumber * numberForEachThread > 10000) {
                throw new Exception("数据量太大，总文档数无法索引");
            }

            while (cycle > 0) {
                int beforeNum = get(indexName);
                System.out.println("插入前，" + indexName + "有文档：" + beforeNum);


                int expectedNum = beforeNum + threadNumber * numberForEachThread;
                int max_loss = 100;


                CountDownLatch countDownLatch = new CountDownLatch(threadNumber);

                ExecutorService executorService = Executors.newFixedThreadPool(threadNumber);

                System.out.println("开始时间：" + new Date());


                for (int i = 0; i < threadNumber; ++i) {
                    executorService.submit(
                            new MongoInsert(countDownLatch, numberForEachThread, "thread_" + i, myCollection)
                    );
                }


                countDownLatch.await();
                Date endInsertDate = null;
                System.out.println("结束时间：" + (endInsertDate = new Date()));

                int currentNum = 0;
                while ((currentNum = get(indexName)) < expectedNum) {
                    System.out.println(new Date() + "  currentNum:" + currentNum);
                    try {
                        TimeUnit.SECONDS.sleep(1);
                    } catch (Exception e) {
                        e.printStackTrace();
                    }
                }

                Date endSyncDate = null;
                System.out.println((endSyncDate = new Date()) + "  currentNum:" + currentNum);

                System.out.println("从插入结束到同步完毕耗时(ms)：" + (endSyncDate.getTime() - endInsertDate.getTime()));

                --cycle;
            }


        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            if (null != mongoClient) {
                mongoClient.close();
            }

        }


    }


    static class MongoInsert implements Runnable {
        private final CountDownLatch countDownLatch;
        private final int numbers;
        private final String threadName;
        private final MongoCollection<Document> collection;

        public MongoInsert(CountDownLatch countDownLatch, int numbers, String threadName, MongoCollection<Document> collection) {
            this.countDownLatch = countDownLatch;
            this.numbers = numbers;
            this.threadName = threadName;
            this.collection = collection;
        }

        @Override
        public void run() {

            try {
                for (int i = 0; i < this.numbers; ++i) {
                    Document document = new Document();
                    document.append("threadName", this.threadName);
                    document.append("order", i);
                    this.collection.insertOne(document);
                }

            } catch (Exception e) {
                e.printStackTrace();
            } finally {
                this.countDownLatch.countDown();
            }


        }
    }
}

```

#### 2.2.5.3 优化monstache部署方式及参数
使用上面的方式对比插入完毕的时间和change stream收到的时间，分片集群部署的MongoDB的change stream的延迟达到7~9秒，再结合前面提到的官方文档，考虑不从分片集群的router server读取change stream，而是直接从各个分片直接读取，省略了router的聚合排序步骤。

最终，每个分片对应的monstache配置如下：

```
#mongo-url = "mongodb://xx.xx.xx.xx:31000"

#这是每个分片的连接串
mongo-url = "mongodb://cloud100:30001,cloud101:30001,cloud102:30001"

#mongo-url = "mongodb://cloud100:30002,cloud101:30002,cloud102:30002"
#mongo-url = "mongodb://cloud100:30003,cloud101:30003,cloud102:30003"

mongo-config-url="mongodb://xx.xx.xx.xx:32000,xx.xx.xx.xx:32000,xx.xx.xx.xx:32000"

#需要多worker模式才需要填写
#workers = ["1", "2", "3"]

elasticsearch-urls = ["http://xx.xx.xx.xx:9200"]

#以下参数字很关键，关系到的数据同步延迟：
#被注释掉的表示使用默认的参数比较不错
#elasticsearch-max-seconds表示：monstache与ES的每个连接，最多等待1秒钟，就必须要要与ES进行数据同步了，不管积攒的待同步的文档数量（#elasticsearch-max-docs）还是文档的总大小（#elasticsearch-max-bytes）有没有达到上限

elasticsearch-max-seconds = 1
#elasticsearch-max-conns = 1
#elasticsearch-max-bytes = 1
#elasticsearch-max-docs = 1

change-stream-namespaces = ["sharding_db_test.jeason_test3"]

#关闭堆栈跟踪（设为false），可提高monstache性能
verbose = true

direct-read-namespaces = ["harding_db_test.jeason_test3"]

#gzip = true

exit-after-direct-reads = false

cluster-name = "cluster-name"

resume = true

resume-name = "resume-name"

[[script]]
namespace = "sharding_db_test.jeason_test"
routing = true
script = """
module.exports = function(doc) {
    doc._meta_monstache = { routing: doc.work_space_id };
    return doc;
}
"""
```
