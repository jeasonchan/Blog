#  1 前言

参考文章：

Mybatis分页插件PageHelper的配置和使用方法      https://www.cnblogs.com/kangoroo/p/7998433.html

Mybatis分页插件PageHelper简单使用         https://www.cnblogs.com/ljdblog/p/6725094.html

Mybatis分页插件PageHelper     https://www.jianshu.com/p/50fcd7f127f0

# 2 常见的分页查方法

在实际的项目开发中，常常需要使用到分页，分页方式分为两种：前端分页和后端分页。

* 前端分页

一次ajax请求数据的所有记录，缓存在浏览器中，利用浏览器的count和分页逻辑，一般前端组件(例如dataTable)会提供分页动作。
特点是：简单，很适合小规模的web平台；当数据量大的时候会产生性能问题，在查询和网络传输的时间会很长。
后端分页
在ajax请求中指定页码pageNum和每页的大小pageSize，后端查询出当页的数据返回，前端只负责渲染。
特点是：复杂一些；性能瓶颈在MySQL的查询性能，这个当然可以调优解决。一般来说，开发使用的是这种方式。
