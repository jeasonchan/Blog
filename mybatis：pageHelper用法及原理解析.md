#  1 前言

参考文章：

Mybatis分页插件PageHelper的配置和使用方法      https://www.cnblogs.com/kangoroo/p/7998433.html

Mybatis分页插件PageHelper简单使用         https://www.cnblogs.com/ljdblog/p/6725094.html

Mybatis分页插件PageHelper     https://www.jianshu.com/p/50fcd7f127f0

# 2 常见的分页查方法

在实际的项目开发中，常常需要使用到分页，分页方式分为两种：前端分页和后端分页。

* 前端分页

一次ajax请求数据的所有记录，缓存在浏览器中，利用浏览器的count和分页逻辑，一般前端组件(例如dataTable)会提供分页动作。

特点是：简单，很适合小规模的web平台；当数据量大的时候会产生性能问题，在查询和网络传输的时间会很长，同时占用

* 后端分页

在ajax请求中指定页码pageNum和每页的大小pageSize，后端查询出当页的数据返回，前端只负责渲染。

特点是：复杂一些；性能瓶颈在MySQL的查询性能，这个当然可以调优解决。一般来说，开发使用的是这种方式。

但是，就算是后端进行分页，也有真分页和假分页。

* 后端假分页

后端的假分页也是一次性查询查询出所有的记录，然后只向前端返回页面的记录，这种假分页查询起来比较费时间，且服务器内存容易爆

* 后端真分页

使用limit a,b 的形式查询，但是随着页数的变大，也会越来越慢，但是一般也不会翻页翻到特别后面……同时还需要额外的使用select count(*)查询总个数

现在，可以直接使用pageHelper了


# 3 真分页查询

## 3.1 不使用插件
在没有使用分页插件的时候需要先写一个查询count的select语句，然后再写一个真正分页查询的语句，MySQL中有对分页的支持，是通过limit子句。

limit关键字的用法是:LIMIT [offset,] rows

offset是相对于首行的偏移量(**首行是0**)，rows是返回条数，limit的语义也就是取符合条件的第offset条到offset+rows-1条。

例如：

```sql
-- 每页5条记录，取第一页，返回的是0~（0+5-1）的记录，也就是返回的是前5条记录
select * from tableA limit 0,5;

-- 每页5条记录，取第二页，返回的是第6条记录，到第10条记录，（number-1）*size，size
select * from tableA limit 5,5;
```

不过当偏移量逐渐增大的时候，查询速度可能就会变慢，性能会有所下降。因为offset要求必须的从头开始慢慢查。

## 3.2 使用插件
PageHelper是一款好用的开源免费的Mybatis第三方物理分页插件，

Github地址:https://github.com/pagehelper/Mybatis-PageHelper

官方地址：https://pagehelper.github.io/

官方的使用文档：https://github.com/pagehelper/Mybatis-PageHelper/blob/master/wikis/zh/HowToUse.md

核心原理：

1、利用ThreadLocal实现LOCAL_PAGE隔离

2、对mybatis的动态sql进行拦截

放一段利用PageHelper进行分页查询的日志：

```
Preparing: SELECT count(0) FROM req_info AS req WHERE req.req_no IN (?, ?) 

Preparing: SELECT serial_id FROM req_info AS req WHERE req.req_no in ( ? , ? ) ORDER BY creation_date DESC limit ?,? 
```

可见，本质上还是利用limit进行分业查询。
