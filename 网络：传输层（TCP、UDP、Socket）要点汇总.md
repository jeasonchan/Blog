# 1 七层网络结构

7层网络结构    https://blog.csdn.net/u010359398/article/details/82142449



# 2 传输层介绍

计算机网络协议(三)——UDP、TCP、Socket      https://blog.csdn.net/ghw15221836342/article/details/100531810?depth_1-utm_source=distribute.pc_relevant.none-task-blog-OPENSEARCH-3&utm_source=distribute.pc_relevant.none-task-blog-OPENSEARCH-3



OSI参考模型——传输层：TCP、UDP协议详解     https://blog.csdn.net/jeffleo/article/details/53967254?depth_1-utm_source=distribute.pc_relevant.none-task-blog-OPENSEARCH-1&utm_source=distribute.pc_relevant.none-task-blog-OPENSEARCH-1





## 2.1 TCP



## 2.2 UDP



## 2.3 Socket



# 3 相关面试题

## 3.1 浏览器输入URL后发什么什么

知乎回答  https://zhuanlan.zhihu.com/p/43369093

在题意不够明确、缺少情景和限定条件的情况下，没法直接作答。在计算机越来越复杂的今天，任何一个条件的变化与组合，都有可能产生千千万万种可能，打破常规。对题目本身而言，就会包括但不仅限于以下几种条件：

* 请求资源类型浏览器类型及版本
* 服务器类型及版本
* 网络协议类型及版本
* 网络链路状况经过哪些中间设备
* 局域网类型及标准物理媒介类型
* 运营商路线

如果请求的是静态资源，那么流量有可能到达 CDN（Content Delivery Network） 服务器；如果请求的是动态资源，那么情况更加复杂，流量可能依次经过代理/网关、Web 服务器、应用服务器、数据库。以下图 阿里云 SLB（Server Load Balancer，负载均衡）高可用部署示意图，它不同于传统的主备切换模式过于依赖单机处理能力，来自公网的请求通过上层交换机的 ECMP（Equal-cost multi-path routing，等价多路径路由）将流量转发给 LVS 集群（四层 SLB），对于 TCP/UDP 请求，LVS 集群直接转发给后端 ECS 集群，对于 HTTP 请求，则转发给 Tengine 集群（七层 SLB），由 Tengine 集群再转发给后端 ECS 集群，集群之间通过实现会话同步、健康检查等机制来保证高可用。

![阿里云负载均衡高可用部署示意图](https://pic1.zhimg.com/80/v2-e2050d15f42336e205122dd51a803230_720w.jpg)

随着业务规模的不断扩大，为了承载千万级甚至亿级流量及海量存储，对系统的多机房容灾、自由扩容的要求越来越高，系统还可能演进为下图的异地多活架构。与传统的灾备设计不同的是，多个数据中心同时对外提供服务，同时保证异地单元间数据库数据的一致性和完整性（CAP 理论）。整个系统架构分为流量层、应用层、数据层，利用 DNS 技术实现 GSLB（Global Server Load Balance，全局负载均衡），实现用户就近访问。如果某个地域的系统发生整体故障，则把所有流量请求切换到另一个地域，来满足异地容灾，这也类似于饿了么现阶段整体架构方案。

![阿里异地多活架构](https://pic3.zhimg.com/80/v2-c7887070dd99ffeb0ea8bf1eb7e58642_720w.jpg)

但是，静态资源不同于动态资源，考虑部署成本和流量成本后，静态资源一般是通过 CDN，利用中间服务器作缓存，如果没有命中缓存，再回源到 OSS（Object Storage Service，阿里云对象存储服务）或者私有服务器。

因此，现实世界的情况是千变万化的，很难想象一个 GET 请求甚至触发银行的转账操作，一切取决于后端实现。重新回到本文的主题，我们排除一切特殊条件，把问题简化一下，如果仅仅考虑：

* 一个 Chrome 浏览器
* 一台 Linux 服务器
* 发起 HTML 请求
* 不考虑任何缓存和优化机制
* 采用 HTTP/1.1 + TLS/1.2 + TCP 协议

这个过程如下：

3.1.1 DNS解析过程
//


3.1.2 HTTP请求过程
//




3.1.3 建立连接
//


3.1.4 发送 HTTP 请求
//



3.1.5 返回 HTTP 响应
//



3.1.6 维持连接
//

3.1.7 断开连接
//


3.1.8 浏览器解析过程
//


3.1.9 主流程
//



3.1.10 渲染流程
//


3.1.11 页面生命周期
//






