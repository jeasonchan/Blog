# 1 前言
参考文章：

当前读、快照都、MVCC     https://www.cnblogs.com/wwcom123/p/10727194.html

Innodb中的事务隔离级别和锁以及MVCC之间的关系        https://blog.csdn.net/weixin_41563161/article/details/105722700

我们使用事务的目的是保证一坨复杂、有序的sql操作的ACID，事务具有ACID特性。


为了达到各个事务级别的ACID目的，sql用了很多实现来保证数据的一致性，又能保证并发性，比如：
* 快照读和当前读：

快照读：通过mvcc实现，极大提高纯select场景和写场景的效率

当前读：select...lock in share mode (共享读锁)、select...for update、update 、delete、insert  这几个语句无论什么级别下，都是读取最新的、commit之后的数据，并且sql会自动给要操作的数据加锁（正是因为这种机制，才会出现数据库死锁），保证DML语句是串行修改数据的

* 行锁、表锁、间隙锁、next-key锁：

为了配合当前读的sql，保证写操作的串行执行而设计的悲观锁

* mvcc：

为了提高读写并发而引入的机制，并不能提高写写冲突效率，还是要靠锁。常说的，非锁定一致性读，就是靠mvcc的比较版本号的机制实现的，没用锁就能实现可重复读。对于主从部署的mysql，从库只处理外部的读，写由binlog的恢复线程执行，mvcc非常适合从库的工作状态。

# 2 当前读
有:

select...lock in share mode (共享读锁)

select...for update

update , delete , insert

当前读,**读取的是最新commit的版本, 并且对读取的记录加锁, 阻塞其他事务同时改动相同记录，避免出现安全问题。**

例如，假设要update一条记录，但是另一个事务已经delete这条数据并且commit了，如果mysql自动不加锁就会产生冲突。所以update的时候肯定要是当前读，得到最新的信息并且锁定相应的记录。

再举个不强制当前读会导致业务错误的例子：

很多开发者误以为将SELECT放入事务，将结果作为判断条件或者写入条件是安全的，其实根据隔离级别不同，是不一定的，举个例子：
```sql
SELECT users表某个用户等级信息，如果是钻石会员，则为他3倍积分

update 将算出的积分UPDATE到user_scores表
```
将这两条语句放入事务也不一定是安全的，这取决于事务的实现，如果是InnoDB的Repeatable Read级别，那么这个事务是不安全的，单纯的select操作用的是mvcc机理，只能读到的小于等于自身事务ID的结果，然而，想用这个可能过期的值用于进行update这样的当前读操作，极有可能导致错误！！！比如，在UPDATE之前，其他事务可能就已经修改了user的等级信息，他可能已经不满足3倍积分条件，而此时再去UPDATE user_scores表，这个事务是个业务不安全的事务。

因此，优化方式是，可以用where合并为一句，也可以，在事务内使用当前读的select for update进行锁定，再update。当然是推荐，一条语句搞定。

## 2.1 next-key锁(行记录锁+Gap间隙锁)

根据where条件里的过滤条件，mysql自动加锁的范围也会变化：

1、主键或唯一索引作为过滤条件（**不同的条目，主键和唯一索引不会重复**），如果当前读时，where条件全部精确命中(=或者in)，只会过滤出单条条目，这种场景本身就不会出现幻读，所以只会加行记录锁。

2、没有索引的列，当前读操作时，会加全表gap锁，生产环境要注意。

3、非唯一索引列，如果where条件部分命中(>、<、like等)或者全未命中，则会加附近间隙锁。例如，某表数据如下，非唯一索引2,6,9,9,11,15。如果要操作非唯一索引列9的数据，间隙锁将会锁定的列是(6,11]，该区间内无法插入数据。

**间隙锁：**

只有在Read Repeatable、Serializable隔离级别才有，就是锁定范围空间的数据，假设id有3,4,5，锁定id>3的数据，是指的4，5及后面的数字都会被锁定，因为此时如果不锁定**没有的数据**，例如当加入了新的数据id=6，就会出现幻读，间隙锁避免了幻读

# 3 快照读
单纯的select操作，不包括上述 select ... lock in share mode, select ... for update。　　　　

Read Committed隔离级别：每次select都生成一个快照读。

Read Repeatable隔离级别：**开启事务后第一个select语句才是快照读的地方，而不是一开启事务就快照读。**

 



# 4 重点小结
1. 事务ID真正产生的时间是事务内第一条select的执行时间，在mvcc中会用来比较版本号，会影响到结果，springboot中基于注解的事务控制，粒度较粗，事务begin和sql执行之间可能会有较大的时间差



# 3 实践
mysql innodb 引擎，假设表 test 初始状态只有如下一条记录：

|id |name|age|
|---|---|---|
|1  |jeason|18|

发生以下相互独立的时序图：

```
假设一：
事务A以可重复读开始，（1）select age from test ——> 
事务B以可重复读开始，（2）update test set age=19 where id=1，commit ——>
事务A（3）select age from test，commit
（3）的结果是：18，mvcc中，事务A只能看到创建ID小于等于自己的ID且删除ID小于等于自己的行

假设二：
事务A以可重复读开始 ——> 
事务B以可重复读开始，（1）update test set age=19 where id=1，commit ——>
事务A，（2）select age from test，commit
（2）的结果是：19，mvcc中，事务A真正获取事务ID的事情是晚于事务B的，因此能看到事务B更新的19

假设三：
事务A以可重复读开始，（1）select age from test ——> 
事务B以可重复读开始，（2）update test set age=19 where id=1，commit ——>
事务A（3）select age from test，（4）update test set age=20 where age=18，（5）select age from test，commit
（5）的结果是：18，当前读的sql语句会读取最新的commit的值（尽管事务ID比自身大），快照读的普通select


假设四：
事务A以可重复读开始，（1）select age from test ——> 
事务B以可重复读开始，（2）update test set age=19 where id=1，commit ——>
事务A（3）select age from test，（4）update test set age=20 where age=19，（5）select age from test，commit
（5）的结果是：20，当前读的sql语句会读取最新的commit的值（尽管事务ID比自身大），快照读的普通select


假设五：
事务A以可重复读开始，（1）select age from test for update ——> 
事务B以可重复读开始，（2）update test set age=19 where id=1，commit ——>
事务A（3）select age from test，（4）update test set age=20 where age=19，（5）select age from test
（5）的结果是：18，（1）执行时会锁住行，（2）根本无法提交，并且，age 没有索引时，会出现锁整个表的间隙锁


假设六：
事务A以可重复读开始，（1）select age from test for update ——> 
事务B以可重复读开始，（2）update test set age=19 where id=1，commit ——>
事务A（3）select age from test，（4）update test set age=20 where age=18，（5）select age from test
（5）的结果是：20，（1）执行时会锁住行，（2）根本无法提交，并且，age 没有索引时，会出现锁整个表的间隙锁
```
