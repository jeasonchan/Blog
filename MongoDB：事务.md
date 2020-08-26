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

## 3.2 事务级别写关注

https://docs.mongodb.com/master/core/transactions/#transactions-write-concern



## 3.3 事务级别读关注

先看非事务中的读关注，再看事务中的读关注。

### 3.3.1 读关注
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


#### 3.3.1.6 非事务和事务中的默认级别
https://docs.mongodb.com/master/reference/mongodb-defaults/


### 3.3.2 事务中的读关注

https://docs.mongodb.com/master/core/transactions/#transactions-read-concern

事务必须依托于一个会话，在会话中开启一个事务后，在事务中进行读操作都会使用事务级别的读关注，和普通的读关注有一些差别（比如，事务中支持的读关注级别只有3个）。

注意！！！！！any read concern set at the collection and database level is ignored inside the transaction.

通过驱动，程序员可以在事务开始时隐式/显式地设置当前事务中读关注级别：

* 如果没显式设置事务级别的读关注级别，就默认使会话级别的读关注级别

* 如果事务和会话级别的读关注级别都没设置




## 3.4 事务级别读首选项

https://docs.mongodb.com/master/core/transactions/#transactions-read-preference






## 3.5 事务和原子性


## 3.6 事务和操作


##### 99999 分片副本集

Production Considerations (Sharded Clusters)    https://docs.mongodb.com/manual/core/transactions-sharded-clusters/#arbiters
