# 1 前言

参考文章：

使用拦截器统一处理异常   https://blog.csdn.net/yipiankongbai/article/details/106740964

本质上就是以一种快捷的方式，定义了一个切面，切@RestController或者@Controller注解，以around的方式捕获异常，然后用统一异常里的异常处理器封装并返回。同时，这也要求了整个项目有统一一致的的返回结果封装。
