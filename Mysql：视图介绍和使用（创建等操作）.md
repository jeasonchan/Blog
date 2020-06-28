# 1 前言

参考文章：

MySQL数据库视图：视图定义、创建视图、修改视图    https://blog.csdn.net/moxigandashu/article/details/63254901

之前学习关联查询join时，发现很多公司都不建议使用关联查询和子查询了，**尤其是子查寻，会有建立、销毁临时表的额外开销**，则这是看到有人建议可以使用视图，好奇学习一下视图。

百度百科：

视图是指计算机数据库中的视图，**是一个虚拟表，其内容由查询定义**。同真实的表一样，视图包含一系列带有名称的列和行数据。但是，**视图并不在数据库中以存储的数据值集形式存在**。行和列数据来自由定义视图时的查询所使用的表，并且在引用视图时**动态生成**。

# 2 视图相关的mysql指令

| 操作指令 | 代码示例  |
| ---     |  ---     |
| 创建视图 | CREATE VIEW 视图名(列1，列2...) AS SELECT (列1，列2...) FROM ...;|
| 使用视图 | 	当成表使用就好   |
| 修改视图 | CREATE OR REPLACE VIEW 视图名 AS SELECT [...] FROM [...]; |
| 查看数据库已有视图  |	SHOW TABLES [like...];（可以使用模糊查找） |
| 查看视图详情 | DESC 视图名或者SHOW FIELDS FROM 视图名 |
| 视图条件限制 | [WITH CHECK OPTION] |

# 3 视图究竟是什么

数据库相对应，每次进行查询工作，都需要编写查询代码进行查询；而将查询代码作为视图创建后，就不必每次都重新编写查询的SQL代码，而是通过视图直接查询即可。视图的作用有**一点点**像存储过程，但是！！！

视图只不过是存储在mysql上的select语句罢了，当对视图请求时，mysql会像执行一句普通的select语句那样的执行视图的select语句，它的性能并不像人们想象得那么出色。 

另外，存储过程在编译后可以生成执行计划，这使得每次执行存储过程的时候效率将会更高，这是存储过程，另外在提交参数的时候，使用存储过程将会减少网络带宽流量（因为只需要传输存储过程中待填写的参数，并必须要传输整个sql语句），这是存储过程相对于普通的sql语句在性能上的最大的优势。

**视图是虚拟表，本身不存储数据，只存储了select语句。**

# 4 创建、修改、查看视图
## 4.1 创建视图
创建视图的代码为：

```sql
CREATE VIEW 视图名(列1，列2...)
AS SELECT (列1，列2...)
FROM ...;
```

可以看到，创建视图和查询相比，增加了前面的CREATE VIEW 视图名 AS。再比如：

假设有一个的复杂的查询sql，需要用到联合查找，把这个的联合查找直接存为视图：

```sql
create view v_order(column1,column2,column3) as
select tb1.column1 tb2.comlumn3 tb2.column3 
from table1 as tb1 
    inner join table2 as tb2
    on tb1.column3=tb2.column3
```

## 4.2 视图运用
使用视图和使用表完全一样，只需要把视图当成一张表就OK了。视图是一张虚拟表。

```sql
-- 尝试查询的上面刚刚创建的视图 v_order
select * from v_orader where column1=2333
```

## 4.3 修改视图

修改视图就是，当有同名视图已经存在则replace原来的视图，没有则新建一个，使用如下命令：

```sql
CREATE OR REPLACE VIEW 视图名 AS SELECT [...] FROM [...];
```

比如：

```sql
create or replace view v_order(column1,column2,column3) as
select tb1.column1 tb2.comlumn3 tb2.column3 
from table1 as tb1 
    inner join table2 as tb2
    on tb1.column3=tb2.column3
```

## 4.4 查看视图字段详情
通过show tables;反馈得到所有的表和视图。同样的，我们可以通过模糊检索的方式专门查看视图，这个时候，视图的命令统一采用"v_视图名"的优势就体现出来了。

比如：

```sql

show tables;

show tables like 'v_%';

DESC 视图名;

-- 或者

SHOW FIELDS FROM 视图名;
```

# 5 视图与数据变更
## 5.1 原表数据变化的会实时影响视图的结果
这点毋庸置疑，不细讲了。

## 5.2 通过视图修改原表的结果
视图不仅可以用来查询试图，甚至可以直接通过视图修改引用的表，但是！！！当且仅当通过视图修改的数据只来自一个表时，才能修改成功；视图来源>=2个表时，却想通过视图修改多个表中的数据时，会报错。

即，**不能跨表修改数据**，因为多个表可能是通过join联结到一起的，跨表修改时，可能不会修改某些字段。

## 5.3 WITH CHECK OPTION
如果在创建视图的时候指定了“WITH CHECK OPTION”，那么更新数据时不能插入或更新不符合视图限制条件的记录。

即，加了“WITH CHECK OPTION”后，能通过视图修改的记录，都必须满足select的条件。

比如：

```sql
-- name这个列可以省略，就默认使用select出来的列名
create view v_view1(name,age) as
select name,age from table1 where age>18
with check option;

-- name这个列可以省略，就默认使用select出来的列名
create view v_view2(name,age) as
select name,age from table1 where age>18;
```

当对以上两个视图插如一下数据时：

```sql
-- 更新不成功，16不满足age>18条件
-- 且该视图有 with check option
insert into v_view1 values('jeason',16);

-- 更新成功，视图不会检查
insert into v_view2 values('jeason',16);
```

**没有特殊的理由，建议加上“WITH CHECK OPTION”命令。**
