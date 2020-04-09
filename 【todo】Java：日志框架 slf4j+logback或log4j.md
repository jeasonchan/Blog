# 1 前言
参考文章：
为什么阿里巴巴禁直接使用日志系统Log4j和Logback中的api？  https://mp.weixin.qq.com/s/cpc7nNYDvBIkpSXqHW09pg
Log4j和slf4j      https://blog.csdn.net/qq_39691226/article/details/80428623

要点：
1. 门面模式，解耦，后期更具体的日志实现更加方便
2. slf4j某些时候的性更更好，不再鼓励的字符串的拼接记录日志
