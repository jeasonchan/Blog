# 1概述
针对某些关键字尽兴练习：
* identity

# 2关键字
先通过sql代码准备好实践的数据库：
```sql
create table runoob(
    id int identity (1,1),
     primary key(id),
    name  varchar(20),
    homepage varchar(100),
    alexa smallint,
    country varchar(10)
);

-- 尽管ID是自增，但是，主动插入特定值，还是能赋值成功
insert into RUNOOB values ( 1,'Google','www.google.com',1,'USA' );

--尝试一下，不插入的数值，缺一位，看能否成功
-- sql执行失败，因为赋值的个数不对
-- insert into RUNOOB values ( 'Google','www.google.com',1,'USA' );

--将自增的值赋值为null，就能实现数据库自动维护
insert into RUNOOB values ( null,'淘宝','www.taobao.com',13,'CN' );
insert into RUNOOB values ( null,'菜鸟教程','www.runoob.com',4689,'CN' );
insert into RUNOOB values ( null,'微博','weibo.com',20,'CN' );
insert into RUNOOB values ( null,'Facebook','www.facebook.com',3,'USA' );
```


## 2.1 identity
标识列， identity(a,b)，ab均为正整数，a表示开始数，b表示增幅。在创建表或者增加列的时候，可进行该约束定义，代码实践如下：
```sql
create table runoob(
    id int identity (1,1),
     primary key(id),
    name  varchar(20),
    homepage varchar(100),
    alexa smallint,
    country varchar(10)
);
-- 或者
alter table RUNOOB add column test int identity (1,1);

```
## 2.2 select

## 2.3 select distinct
