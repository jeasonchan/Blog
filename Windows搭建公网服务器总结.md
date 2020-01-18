# 1 背景
之前买了一年的89的阿里云，配置是1Core 2g ram，不不装什么的中间件的话，应该是够用了。但是！redis、kafka、zookeeper什么的都还没折腾过，怎么可能放弃折腾呢……于是，想以自己的笔记本硬件作为运行环境，探索一套可行的提到方案。最终，终于实现了：
* 有公网IP的前提下
* 在windows中安装vmware workstation（非免费版）
* 通过设置NAT和防火墙
* 实现了，ssh外网访问vmware中的Ubuntu server和搭建的服务
# 2 