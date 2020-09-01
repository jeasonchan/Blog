# 1 背景
部署mongodb时一般是PSS部署架构，即为一主多层的部署方式，最近了解一下PSA部署方式，即为主、从、仲裁三种节点，关于PSA的部署方式，有很多东西需要验证，尤其是PSA部署模式下的读相关、写相关的事务。

参考文档：

**PSA相关：**

MongoDB-PSA架构配置   https://www.jianshu.com/p/3bff8dd0113d

有坑勿踩（一）：MongoDB PSS vs PSA    https://segmentfault.com/a/1190000016993056

MongoDB-副本集PSA架构搭建配置手册    https://blog.csdn.net/xiaocong66666/article/details/82768603

**事务相关：**

官方文档  https://docs.mongodb.com/master/core/transactions/


# 2 什么是PSA



# 3 事务
mongodb对于单文档的操作具原子性，具有单文档的事物性。但是，数据库中使用事物时一般都是同时操作多条记录（在mongo里面也就是同时操作多个文档对象），老版本的mongo不支持多文档的事务时，需要程序员自己写代码实现多文档事务的ACID。目前，较新版本的mongo已经支持跨分片、副本集、库、集合的多文档事务（包含分布式事务）。

snapshot:

MongoDB database backups called snapshots and they are stored in containers called snapshot stores. 对于分片集群，每个分片其实都是一个副本集，所有的分片的**同一时刻(T时刻)**snapshot拼起来才算整个分片集群在T时刻的snapshot。


## 3.1 API 
* 使用的mongo驱动必须和mongodb版本相匹配，尤其是针对副本集和分片副本集的事务操作。

* 事务中的每一个操作动作都必须和session关联，也就是，mongodb的事务是。

* 事务中的增删改查用的是 transaction-level read concern, transaction-level write concern 和 transaction-level read preference

* mongodb 4.2版本前，事务中的操作必须针对已经存在的集合，因为在事务中不允许创建新的集合，而一般情况下，mongodb的集合会自动创建；从4.4版本开始，已经允许显示或者隐式创建集合和索引

### 3.1.1 Java同步api示例
```java
/*
  For a replica set, include the replica set name and a seedlist of the members in the URI string; e.g.
  String uri = "mongodb://mongodb0.example.com:27017,mongodb1.example.com:27017/admin?replicaSet=myRepl";
  For a sharded cluster, connect to the mongos instances; e.g.
  String uri = "mongodb://mongos0.example.com:27017,mongos1.example.com:27017:27017/admin";
 */

final MongoClient client = MongoClients.create(uri);

/*
    Create collections.
 */

client.getDatabase("mydb1").getCollection("foo")
        .withWriteConcern(WriteConcern.MAJORITY).insertOne(new Document("abc", 0));
client.getDatabase("mydb2").getCollection("bar")
        .withWriteConcern(WriteConcern.MAJORITY).insertOne(new Document("xyz", 0));

/* Step 1: Start a client session. */

final ClientSession clientSession = client.startSession();

/* Step 2: Optional. Define options to use for the transaction. */

TransactionOptions txnOptions = TransactionOptions.builder()
        .readPreference(ReadPreference.primary())
        .readConcern(ReadConcern.LOCAL)
        .writeConcern(WriteConcern.MAJORITY)
        .build();

/* Step 3: Define the sequence of operations to perform inside the transactions. */

TransactionBody txnBody = new TransactionBody<String>() {
    public String execute() {
        MongoCollection<Document> coll1 = client.getDatabase("mydb1").getCollection("foo");
        MongoCollection<Document> coll2 = client.getDatabase("mydb2").getCollection("bar");

        /*
           Important:: You must pass the session to the operations.
         */
        coll1.insertOne(clientSession, new Document("abc", 1));
        coll2.insertOne(clientSession, new Document("xyz", 999));
        return "Inserted into collections in different databases";
    }
};
try {
    /*
       Step 4: Use .withTransaction() to start a transaction,
       execute the callback, and commit (or abort on error).
    */

    clientSession.withTransaction(txnBody, txnOptions);
} catch (RuntimeException e) {
    // some error handling
} finally {
    clientSession.close();
}
```

### 3.1.2 C++11示例
```cpp
// The mongocxx::instance constructor and destructor initialize and shut down the driver,
// respectively. Therefore, a mongocxx::instance must be created before using the driver and
// must remain alive for as long as the driver is in use.
mongocxx::instance inst{};

// For a replica set, include the replica set name and a seedlist of the members in the URI
// string; e.g.
// uriString =
// 'mongodb://mongodb0.example.com:27017,mongodb1.example.com:27017/?replicaSet=myRepl'
// For a sharded cluster, connect to the mongos instances; e.g.
// uriString = 'mongodb://mongos0.example.com:27017,mongos1.example.com:27017/'
mongocxx::client client{mongocxx::uri{"mongodb://localhost/?replicaSet=replset"}};

write_concern wc_majority{};
wc_majority.acknowledge_level(write_concern::level::k_majority);

read_concern rc_local{};
rc_local.acknowledge_level(read_concern::level::k_local);

read_preference rp_primary{};
rp_primary.mode(read_preference::read_mode::k_primary);

// Prereq: Create collections.

auto foo = client["mydb1"]["foo"];
auto bar = client["mydb2"]["bar"];

try {
    options::insert opts;
    opts.write_concern(wc_majority);

    foo.insert_one(make_document(kvp("abc", 0)), opts);
    bar.insert_one(make_document(kvp("xyz", 0)), opts);
} catch (const mongocxx::exception& e) {
    std::cout << "An exception occurred while inserting: " << e.what() << std::endl;
    return EXIT_FAILURE;
}

// Step 1: Define the callback that specifies the sequence of operations to perform inside the
// transactions.
client_session::with_transaction_cb callback = [&](client_session* session) {
    // Important::  You must pass the session to the operations.
    foo.insert_one(*session, make_document(kvp("abc", 1)));
    bar.insert_one(*session, make_document(kvp("xyz", 999)));
};

// Step 2: Start a client session
auto session = client.start_session();

// Step 3: Use with_transaction to start a transaction, execute the callback,
// and commit (or abort on error).
try {
    options::transaction opts;
    opts.write_concern(wc_majority);
    opts.read_concern(rc_local);
    opts.read_preference(rp_primary);

    session.with_transaction(callback, opts);
} catch (const mongocxx::exception& e) {
    std::cout << "An exception occurred: " << e.what() << std::endl;
    return EXIT_FAILURE;
}

return EXIT_SUCCESS;
```

## 3.2 写关注


### 3.2.1 非事务的写关注

https://docs.mongodb.com/master/reference/write-concern/



Write concern describes the level of acknowledgment requested from MongoDB for write operations to a standalone [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) or to [replica sets](https://docs.mongodb.com/master/replication/) or to [sharded clusters](https://docs.mongodb.com/master/sharding/). In sharded clusters, [`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos) instances will pass the write concern on to the shards. 也就是，写相关级别就是写入的确认消息的返回时机。



从4.4开始，副本集和分片集群也开始支持设置全局默认的 写相关等级，没有明确写相关等级的都会使用全局默认写相关等级。如果全局的也没有设置，写相关等级看3.4章节的内容。



写相关等级的可通过以下配置对象进行配置：

```json
{ w: <value>, j: <boolean>, wtimeout: <number> }
```

也就是三个参数，通过对这三个参数的设置实现不同等级的写关注等级。

-  [w](https://docs.mongodb.com/master/reference/write-concern/#wc-w) ：写操作已经传播到的mongod数量
-  [j](https://docs.mongodb.com/master/reference/write-concern/#wc-j) ：写操作是否已经写到[on-disk journal](https://docs.mongodb.com/master/core/journaling/)
- [wtimeout](https://docs.mongodb.com/master/reference/write-concern/#wc-wtimeout) ：超时时长，便面写操作一直阻塞业务线程

分别对三个参数进行介绍。

#### 3.2.1.1 w

w: <value>，其中的value参数有三种填写类型。

- 直接使用数字

数字1，对于单节点mongo，得到该单节点确认就返回;对于多节点的mongodb，首先，写操作必须是直接针对primary节点进行，得到多节点的primary节点的确认就返回。对于多节点的部署，如果主节点先返回确认然后就立刻挂了，写入操作还没有来得及传播到任何从节点，写入的数据会**回滚**，毕竟，晋升为主节点的从节点根本没有收到刚写入的数据。

对于比1大的数字N，剩余的（N-1）个需要从节点的确认返回。



- 字符串"majority"

 mongodb会根据 [calculated majority](https://docs.mongodb.com/master/reference/write-concern/#calculating-majority-count)和存储数据且具有投票资格的mongodb节点数，计算出一个的数字。

比如，对于典型的PSS部署，计算出的数字就是2，也就是该情况下的"majority"等效于2。



- 自定义的tag



#### 3.2.1.1 j

[on-disk journal](https://docs.mongodb.com/master/core/journaling/)，和mysql的redo log类似，mongodb有种write-ahead log，write-ahead log是WiredTiger Storage Engine产生的，在硬盘中持久化的数据操作记录;同时还有一种内存型的，in-Memory Storage Engine。

对于WiredTiger Storage Engine，j为true时，**有w参数数量的mongodb节点将写操作已经写到write-ahead log中才返回确认**。

对于in-Memory Storage Engine，无论j为true还是false，都是立即返回。

`j: true` 无法保证，没来得及传播到从节点的数据，在主机点宕机后，没来得及传播的数据会出现在新晋主节点上。



#### 3.2.1.1 wtimeout

单位毫米，有效的值必须>=1，为0等或者不写等同于无限期阻塞。



写操作只会等待wtimeout毫秒的时间，写出操作超时后返回error后，写操作其实还会继续进行，结果可能是最终失败或者成功。最终成功的就成功了，最终失败的（比如，写入的个数达不到w的限制，w限制传播到10，结果一共10节点，挂了1个，只传播到9个），失败的会进行回滚（已经写入的9个节点的数据会回滚）。





### 3.2.2 事务的写关注
https://docs.mongodb.com/master/core/transactions/#transactions-write-concern

在事务中的 写相关 的操作的，会使用事务级别的 写相关 等级进行数据提交。

在事务中，绝对不能为某一个单独的写操作设置 写相关 等级，而是应该使用开启事务时声明的那个 写相关  事务等级。

在开启事务时，设置事务的  写相关  等级：

*  事务的 写相关 等级没设置，就默认使用会话的 写相关 等级

* 如果事务和会话的 写相关  等级都没有设置，就是用客户端级别的 写相关 等级，客户端的  写相关  等级是 w:1 级别


#### 3.2.2.1 事务写相关等级w：1

* Write concern w: 1 returns acknowledgement after the commit has been applied to the primary. BUT When you commit with w: 1, your transaction can be rolled back if there is a failover.

* When you commit with w: 1 write concern, transaction-level "majority" read concern provides no guarantees that read operations in the transaction read majority-committed data.

* When you commit with w: 1 write concern, transaction-level "snapshot" read concern provides no guarantee that read operations in the transaction used a snapshot of majority-committed data.

注意点！！！！！只要事务中写相关的级别是w：1，就算读相关级别是majority或者snapshot，都无法保证读得是该级别应该读到的数据……所以，如果对读取数据的有效性有要求，事务级别的 写相关  等级 基本就不能 w：1 等级了。


#### 3.2.2.2 事务写相关等级w：majority

* 对数据库返回的消息的影响：Write concern w: "majority" returns acknowledgement after the commit has been applied to a majority (M) of voting members; i.e. the commit has been applied to the primary and (M-1) voting secondaries.

* 对majority读级别的影响：When you commit with w: "majority" write concern, transaction-level "majority" read concern guarantees that operations have read majority-committed data. For transactions on sharded clusters, this view of the majority-committed data is not synchronized across shards.

* 对snapshot读级别的影响：When you commit with w: "majority" write concern, transaction-level "snapshot" read concern guarantees that operations have from a synchronized snapshot of majority-committed data.

## 3.3 读关注

先看非事务中的读关注，再看事务中的读关注。

### 3.3.1 非事务的读关注
https://docs.mongodb.com/master/reference/read-concern/

read Concern，就是从mongodb读取数据时使用的隔离级别，通过指定读关注的隔离级别来控制并发性和可用性，比如，读关注的级别是能读到就行的级别，那并发性就非常高，可用性也会很高；如果要求读关注级别必须是所有的配置过的节点都读一遍，那么响应时间变长（并发性降低），并且有一个节点发生了故障，那么读取就会失败（因为，有一个就节点数据没读到），也就是降低了mongodn集群的高可用性。

从3.2开始，支持指定以什么级别的读关注级别读取。

从4.4开始，支持为副本集和分片集群设置全局性的默认的读关注级别。设置方法，见https://docs.mongodb.com/master/reference/command/setDefaultRWConcern/#dbcmd.setDefaultRWConcern


mongodb支持的读关注级别有：local、available、majority、linearizable、snapshot

#### 3.3.1.1 local
描述：	

The query returns data from the instance with no guarantee that the data has been written to a majority of the replica set members (i.e. may be rolled back).


默认的使用场景：


可以使用的场景：


详见：

https://docs.mongodb.com/master/reference/read-concern-local/#readconcern.%22local%22

#### 3.3.1.2 available

详见：

https://docs.mongodb.com/master/reference/read-concern-available/#readconcern.%22available%22

#### 3.3.1.3 majority

详见：

https://docs.mongodb.com/master/reference/read-concern-majority/#readconcern.%22majority%22

#### 3.3.1.4 linearizable


详见：

https://docs.mongodb.com/master/reference/read-concern-linearizable/#readconcern.%22linearizable%22

#### 3.3.1.5 snapshot

详见：

https://docs.mongodb.com/master/reference/read-concern-snapshot/#readconcern.%22snapshot%22


### 3.3.2 事务中的读关注

https://docs.mongodb.com/master/core/transactions/#transactions-read-concern

事务必须依托于一个会话，在会话中开启一个事务后，在事务中进行读操作都会使用事务级别的读关注，和普通的读关注有一些差别（比如，事务中支持的读关注级别只有3个）。

注意！！！！！any read concern set at the collection and database level is ignored inside the transaction.

通过驱动，程序员可以在事务开始时隐式/显式地设置当前事务中读关注级别：

* 如果没显式设置事务级别的读关注级别，就默认使会话级别的读关注级别

* 如果事务和会话级别的读关注级别都没设置，就会默认使用客户端级别的读关注级别（也就是"local" for reads against the primary.）


前文说到，事务中能够使用的读关注级别的比正常的要少，一共只有三个：local、majority、snapshot

#### 3.3.2.1 事务级别的local

* Read concern "local" returns the most recent data available from the node but can be rolled back.

* For transactions on sharded cluster, "local" read concern cannot guarantee that the data is from the same snapshot view across the shards. If snapshot isolation is required, use "snapshot" read concern.

* Starting in MongoDB 4.4, with feature compatibility version (fcv) "4.4" or greater, you can create collections and indexes inside a transaction. If explicitly creating a collection or an index, the transaction must use read concern "local". Implicit creation of a collection can use any of the read concerns available for transactions.


#### 3.3.2.2 事务级别的majority

* Read concern "majority" returns data that has been acknowledged by a majority of the replica set members (i.e. data cannot be rolled back) if the transaction commits with write concern “majority”.

* If the transaction does not use write concern “majority” for the commit, the "majority" read concern provides no guarantees that read operations read majority-committed data.

* For transactions on sharded cluster, "majority" read concern cannot guarantee that the data is from the same snapshot view across the shards. If snapshot isolation is required, use "snapshot" read concern.

事务中使用majority级别读到的数据的可靠性，和数据写入时所使用的事物写关注等级强**相关**。

对于分片集群，majority和local根本无法保证各分片数据的一致性。

#### 3.3.2.3 事务级别的snapshot

* Read concern "snapshot" returns data from a snapshot of majority committed data if the transaction commits with write concern “majority”.

* If the transaction does not use write concern “majority” for the commit, the "snapshot" read concern provides no guarantee that read operations used a snapshot of majority-committed data.

* For transactions on sharded clusters, the "snapshot" view of the data is synchronized across shards.

比majority相比没，就多了能保证 the "snapshot" view of the data is synchronized across shards




## 3.4 非事务和事务中的默认读写级别
https://docs.mongodb.com/master/reference/mongodb-defaults/


### 3.4.1 默认的读相关级别
![事务和非事务的读相关级别继承关系](https://docs.mongodb.com/master/_images/read-write-concern-inheritance.bakedsvg.svg)

默认的事务等级和   读操作针对的mongo节点、使用的会话等因素相关。

* 对主节点读，默认是local级别：

能读到可能会被回滚的数据，且不保证数据的因果有序性


* 在有序会话（causally consistent sessions）中，对从节点读，默认是local级别：

能读到可能会被回滚的数据，尽管在有序会话中，但是不保证数据的因果有序性


* 在无序会话中，对从节点读，默认是available级别：

能读到可能会被回滚的数据，尽管在有序会话中，但是不保证数据的因果有序性;

对于被分片的集合，这种读方式甚至还能读取到orphaned documents（孤儿文档，shard平衡过程失败后，同时存在于多个chunk的文档）


可见，在事务中是，对于事务的默认读相关级别，如果session、transaction都没有显式声明事务读相关级别，会默认使用local读相关级别。

### 3.4.2 默认的写相关级别
![事务和非事务的写相关级别继承关系](https://docs.mongodb.com/master/_images/read-write-concern-inheritance.bakedsvg.svg)

直接只有一种默认的写入级别，w:1：

能写入一个就直接返回确认消息，并且写入操作不能保证因果的有序性



### 3.4.3 因果一致性保证（Causally Consistency Guarantees）

通过因果一致性client会话（causally consistent client sessions，https://docs.mongodb.com/master/core/read-isolation-consistency-recency/#sessions），client会话也仅能在以下场景保证读写的因果一致性：

读相关和写相关**都使用**majority级别。

应用程序使用的会话都是client会话，因果一致性保证就是类似于zookeeper能保证所有订阅者收到的操作顺序都是完全一致的，尽管收到的可能会有一些延迟。在某些要求操作具有有序性的场景下，在一个因果有序会话中，应用程序本身也应该只使用一个线程进行读写操作，多线程的读写操作的有序性并非程序员能完美控制的。

与client会话（https://docs.mongodb.com/master/release-notes/3.6/#client-sessions）相对的，还有server session（https://docs.mongodb.com/master/reference/server-sessions/），server session是用来实现因果有序性的底层框架。




## 3.5 事务级别读首选项

https://docs.mongodb.com/master/core/transactions/#transactions-read-preference



## 3.6 事务和原子性


## 3.7 事务和操作



---

# 4 分片副本集中的事务

Production Considerations (Sharded Clusters)    https://docs.mongodb.com/manual/core/transactions-sharded-clusters/#arbiters
