参考文档<https://www.cnblogs.com/wuchaofan1993/p/5833526.html>

新建表：
create table [表名]
(
[自动编号字段] int IDENTITY (1,1) PRIMARY KEY ,
[字段1] nVarChar(50) default \'默认值\' null ,
[字段2] ntext null ,
[字段3] datetime,
[字段4] money null ,
[字段5] int default 0,
[字段6] Decimal (12,4) default 0,
[字段7] image null ,
)

删除表：
Drop table [表名]

插入数据：
INSERT INTO [表名] (字段1,字段2) VALUES (100,\'51WINDOWS.NET\')

删除数据：
DELETE FROM [表名] WHERE [字段名]>100

更新数据：
UPDATE [表名] SET [字段1] = 200,[字段2] = \'51WINDOWS.NET\' WHERE [字段三] = \'HAIWA\'

新增字段：
ALTER TABLE [表名] ADD [字段名] NVARCHAR (50) NULL

删除字段：
ALTER TABLE [表名] DROP COLUMN [字段名]

修改字段：
ALTER TABLE [表名] ALTER COLUMN [字段名] NVARCHAR (50) NULL

重命名表：(Access 重命名表，请参考文章：在Access数据库中重命名表)
sp_rename \'表名\', \'新表名\', \'OBJECT\'

//====================bilibli视频教程代码实践========================

主键，唯一且非空

外键，表示引用了别的表里的主键，可以为空，代表当前实体没有引用
```sql
create table major
(
    mno   int,
    mname varchar(20),
--     设置主键的方式
    primary key (mno)
);

-- windows下，对大小写不敏感，或者数据库配置为对字段大小写不敏感
select *
from MAJOR;

create table stu
(
    sno   int,
    sname varchar(30),
    age   smallint,
    sex   bit,
    mno   int,
    primary key (sno),
--    讲该键指向别的表的主键，本质上表达了ER模型中的R
    foreign key (mno) references MAJOR (mno)
);


select *
from STU;

create table cou
(
    cno     int,
    cname   varchar(30),
    ctime   smallint,
--     浮点数
    ccredit decimal(4, 2),
    primary key (cno)

);

select *
from COU;


create table sc
(
    sno   int,
    cno   int,
    grade decimal(5, 2),
--     将数值对作为主键，也算非空且唯一
    primary key (sno, cno)
);

select *
from sc;

-- 删除一下，补充外键
drop table sc;


create table sc
(
--     学生编号
    sno   int,
--     课程编号
    cno   int,
--  课程成绩
    grade decimal(5, 2),

--     将数值对作为主键，也算非空且唯一
    primary key (sno, cno),
    foreign key (sno) references STU (SNO)
);


-- 对sc表新增一列并设为外键，因为，课程编号也是其他表中的主键
-- 增加一个约束，使其值只能从COU表中的主键取值
alter table SC
    add constraint sc_cno_from_cou foreign key (cno) references COU (CNO);


select * from SC;
--
--
--
-- 哔哩哔哩第二节
-- 给某表增加一列（设定类型是必要的）
alter table STU
    add column qq varchar(20);

-- 查询表中所有的字段
select *
from STU;

-- 删除某表格中的某一个字段
alter table STU
    drop column qq;


select *
from STU;

create table to_be_deleted(
    t int,
    primary key (t)
);

select * from TO_BE_DELETED;

-- 删除表操作
drop table TO_BE_DELETED;

-- 向表中插入一条数据，比如，先增加课程信息
-- 先查看一下表的字段有哪些
select * from MAJOR;
-- 插入数据
insert into MAJOR(MNO,mname) values ( 1,'计算机科学和技术' );
select * from MAJOR;
insert into MAJOR(MNO,mname) values ( 2,'软件工程' );
select * from MAJOR;
-- 这条语句会执行失败，以为MAJOR的主键是数据库操作这自己维护的，每次需要自己插入唯一且非空的值
-- insert into MAJOR(mname) values ( '软件工程' );
-- 查看一下学生表的字段
select * from STU;
-- 插入的时候不明确字段，则表示每条字段都插入
insert into PUBLIC.STU values ( 1,'jeason',12,1,1,'12345678' );
select * from STU;
-- 试图插入一条不不符合约束体条件的数据库，外键无对应的主键
insert into PUBLIC.STU values ( 2,'jeason2',12,1,1,null );
-- 外键写错了，因该构造一条错误的外键，删除该条数据
DELETE FROM PUBLIC.STU WHERE SNO = 2;
-- 再试图插入一条不不符合约束体条件的数据库，外键无对应的主键
-- 发现报错，不满足约束条件
-- insert into PUBLIC.STU values ( 2,'jeason2',12,1,6,null );

```

