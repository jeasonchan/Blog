```sql
# =========函数定义区===========



# ==============================
create database workcontent;

use workcontent;

create table if not exists daily
(
    id            int unsigned auto_increment,
    create_time   date,
    status        tinyint,
    content       text,
    extra_content text,
    primary key (id)
) charset = utf8;

show columns from daily;

alter table daily
    change create_time hahah date;

insert into daily
values (null, now(), 0, '试试', '');

select *
from daily;

alter table daily
    modify status tinyint default 0;

insert into daily
    (create_time, content, extra_content)
values (now(), '试试2', '');

alter table daily
    modify status char(100);
alter table daily
    modify status tinyint default 0;

select content
from daily
where status = 0;

select id, status, content
from daily;

select id
from daily
where status is null;

update daily
set status=0
where status is null;

delete
from daily
where content like '试试%';

alter table daily
    modify extra_content text;

insert into daily (content, extra_content)
values ('试试', '');

alter table daily
    modify create_time datetime default now();

insert into daily (content, extra_content)
values ('试试', '');

update daily
set create_time=now()
where create_time is null;

delete
from daily
where length(content) < 100;

insert into daily (content, extra_content)
values ('1', '');
insert into daily (content, extra_content)
values ('22', '');
insert into daily (content, extra_content)
values ('333', '');
insert into daily (content, extra_content)
values ('4444', ''),
       ('55555', '');

select id, length(content)
from daily;


insert into daily (content, extra_content)
values ('屌屌', '');
insert into daily (content, extra_content)
values ('屌', '');
insert into daily (content, extra_content)
values ('屌1', '');

select id, content, extra_content
from daily
where content like '%1%'
   or extra_content like '%1%';

delete from daily where id=1;

insert into daily (content) values ('验证更客户端之后的Agent接入问题'),
                                   ('取消操作的退出问题');

# 查询当天添加的字段
select * from daily where to_days(now())-to_days(create_time)=0;
# 查看近一个星期的记录
select * from daily where to_days(now())-to_days(create_time)<=2;

update daily set  status=1,extra_content='icener已更新方案' where to_days(now())-to_days(create_time)<=3 and content='取消操作的退出升级模式问题';


create table mvcc_test(
    id int auto_increment,
    name varchar(20),
    primary key(id)
)

select *
from daily;

delete from daily where length(content)<10;

select id,content,extra_content,status from daily;

# 定义当前会话有效的变量
set @isDoneStatus =1;
# 当前会话定义的变量
select @isDoneStatus;

update daily set status=@isDoneStatus;

select * from daily;

update daily set extra_content='已验证通过' where content like '%Agent%' and extra_content is null;

select *
from daily;

select *
from daily where content like '%Agent%';

# 开始一个事务会话
start transaction ;
set session transaction isolation level repeatable read;
select * from mvcc_test;
commit ; # 提交会话

```
