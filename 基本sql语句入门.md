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

