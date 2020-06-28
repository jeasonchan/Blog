@[TOC]
# 1 activiti简介
最近公司项目用到了工作流workflow，虽然还没搞懂前端是如何解析工作流描述的XML文件的，但是，以图形化的方式编辑工作流和调用脚本的方式还是十分长见识的，再结合groovy作为工作流中的脚本语言，能充分使用Java庞大的库，感觉是十分不错的快速流程实现方法。
其中，用到的activiti平台<https://www.activiti.org>就是用到的图形化的工作流编辑软件，该软件采用B/S架构，依赖jrm和tomcat，jrm是运行时环境，这点毋庸置疑，tomcat在这个软件中的作用是服务器和网络容器（处理浏览器的http请求，处理前端的交互数据，dropwizard中自动封装了jetty，因此，dropwizard打包为一个jar包时也能够正常运行）。
# 2 安装
前提：已经使用apt install 安装了jdk
1. 下载并解压得到tomcat的文件夹
2. 进入bin文件夹，在Windows下使用startup.bat启动服务，在Linux下就使用"bash startup.sh"启动，我的提示无法找到xxxx.sh文件，是该脚本文件的权限不足，正常情下，使tomcat中每个文件和文件夹具有755权限即可，bash命令为：
```bash
chmod -R 755  ~/下载/tomcat/    #以迭代遍历的方式，使每个文件及文件夹权限为755
```
再使用startup.sh启动tomcat。浏览器访问 <http://localhost:8080> 可看到tomcat主页，网页后端和servlet全由tomcat提供。

3. 然后再使用shutdown.sh或者shutdown.bat脚本停止服务
4. 下载activiti包，解压，进入wars文件夹，复制仅有的两个war包，并粘贴到tomcat的webapps文件夹下
5. 再次startup tomcat，会发现tomcat已经自动将两个war包解压，并生成相应的文件夹了
6. 通过<http://localhost:8080>可访问tomcat的主页，通过<http://localhost:8080/{webapps中的文件夹名称}>可访问相应的应用，比如访问<	http://localhost:8080/activiti-explorer>即可使用activiti了，用户名和密码均为默认的kermit，同时还可以访问官方提供的另一个war包生成的范例
# 3 简单使用
（……待补充……）
# 4 感受
就像图像编程一样，也有异常抛出什么的，工作流只是对其中执行脚本的一个有序排列、调用，用起来也有一种文本编程的感觉，感觉是很不错的将大量执行脚本有组织的工具。

