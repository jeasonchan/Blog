# 1 前言

参考文章：

MySQL中键(key)和索引(index)的区别   https://blog.csdn.net/amoscykl/article/details/88553308

mysql创建索引的三种办法    https://blog.csdn.net/microopithecus/article/details/94186788

# 2 key和index的区别
很多开源项目，创表的时候只创建各种key（主键primary key简称pri；唯一键 unique key，简称uni；普通的键 key，简称mul）不创建索引；还有的创表时，只创建主键，其余就创建索引。

那mysql的key和index究竟是什么，两个有什么区别？

## 2.1 键（key）
key 是数据库的**物理结构**，它**包含两层意义，一是约束（偏重于约束和规范数据库的结构完整性），二是索引（辅助查询用的）**。类型包括primary key, unique key, foreign key等，作用如下：
  
primary key 有两个作用，一是约束作用（constraint)，用来规范一个存储主键和唯一性，但同时也在此key上建立了一个index；

unique key  有两个作用，一是约束作用（constraint)，规范**数据的唯一性**，但同时也在这个key上建立了一个index；

foreign key  有两个作用，一是约束作用（constraint)，规范数据的引用完整性，但同时也在这个key上建立了一个index；

可见，mysql的key是同时具有constraint和index的意义，**这点和其他数据库表现的可能有区别。（至少在Oracle上建立外键，不会自动建立index）**。

### 2.1.1 创建key的方式

创建key也有如下几种方式：

（1）在创表时，在字段级**以key方式**建立，在定义字段的同时申明为主键，如 create table t (id int not null primary key);

（2）在创表时，在表级**以constraint方式**建立，显式使用CONSTRAINT这个关键字，并且能给这个约束起名字。如create table t(id int, CONSTRAINT pk_t_id PRIMARY key (id));

（3）在创表时，在表级**以key方式**建立，如create table t(id int, primary key (id));

其它key创建类似，但不管那种方式，既建立了constraint，又建立了index，只不过index使用的就是这个constraint或key。

# 2.2 索引（index）

索引同样是数据库的物理结构（索引一般比较大，比如，以B+树的数据结构形式存储在磁盘中），它只是辅助查询的，它**创建时会在另外的表空间（mysql中的innodb表空间）以一个类似目录的结构存储**。索引要分类的话，分为前缀索引、全文本索引等，当然不同的分类方式还有其他类型的索引；因此，**索引只是索引，它不会去约束索引的字段的行为（那是key要做的事情）**。

## 2.1 创建索引的方式

创建索引的方式和键大同小异，如，create table t(id int, index inx_tx_id  (id)); 接下来系统介绍一下如何创建索引，有一下三种方式：

* create table 时直接创建索引
* 单独使用 create index 创建所以
* 使用ALTER TABLE来为表增加索引

**但是，每种方式能创建的索引类型是不同的**

### 2.1.1 ALTER TABLE 创建索引

ALTER TABLE用来创建普通索引、UNIQUE索引或PRIMARY KEY索引。

```sql
ALTER TABLE table_name ADD INDEX index_name (column_list)

ALTER TABLE table_name ADD UNIQUE (column_list)

ALTER TABLE table_name ADD PRIMARY KEY (column_list)

-- FULLTEXT全文索引 
ALTER TABLE `table_name` ADD FULLTEXT ( `column`) 

-- INDEX多列索引
ALTER TABLE `table_name` ADD INDEX index_name ( `column1`, `column2`, `column3` )
```

其中table_name是要增加索引的表名，column_list指出对哪些列进行索引，多列时各列之间用逗号分隔。**索引名index_name可选，缺省时，MySQL将根据第一个索引列赋一个名称**。另外，ALTER TABLE允许在单个语句中更改多个表，因此可以在同时创建多个索引。

 

### 2.1.2 CREATE INDEX 创建索引

CREATE INDEX可对表增加普通索引或UNIQUE索引，**该方式不能创建主键索引**。

```sql
CREATE INDEX index_name ON table_name (column_list)

CREATE UNIQUE INDEX index_name ON table_name (column_list)
```
 
table_name、index_name和column_list具有与ALTER TABLE语句中相同的含义，不能用CREATE INDEX语句创建PRIMARY KEY索引。


### 2.1.3 创表时创建索引

```sql
CREATE TABLE test
(
    hahha   int,
    heheheh int,
    INDEX index_name_可省略 (hahha),
    UNIQUE (heheheh)

);
```

## 2.2 删除索引的方式

可利用ALTER TABLE或DROP INDEX语句来删除索引。类似于CREATE INDEX语句，DROP INDEX可以在ALTER TABLE内部作为一条语句处理，语法如下。**可见，单独删除索引都是要知道索引的名称的。**

```sql
DROP INDEX index_name ON talbe_name

ALTER TABLE table_name DROP INDEX index_name

ALTER TABLE table_name DROP PRIMARY KEY
```

## 2.3 小结
(1) 我们说索引分类，分为主键索引、唯一索引、普通索引(这才是纯粹的index)等，也是基于是不是把index看作了key。比如：

```sql
-- 把index当作了key使用
create table t(id int, unique index inx_tx_id  (id));  
```

(2) 最重要的也就是，不管如何描述，理解index是纯粹的index，还是被当作key，当作key时则会有两种意义或起两种作用。
