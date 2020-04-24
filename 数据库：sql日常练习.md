使用SQL来记录日记：
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

alter table daily modify status char(100);
alter table daily modify status tinyint default 0 ;

select content from daily where status=0;

select id,status,content from daily;

select id from daily where status is null;

update daily set status=0 where status is null;

delete from daily where content like  '试试%';

alter table daily modify extra_content text;

insert into daily (content,extra_content) values ('试试','');

alter table daily modify create_time datetime default now();

insert into daily (content,extra_content) values ('试试','');

update daily set create_time=now() where create_time is null;



```
