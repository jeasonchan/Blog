# 1 背景
docker镜像制作的原理是，在拉取到的公共的镜像的基础上，在运行起来的容器内执行一些bash命令，完成程序依赖建立，最终实现程序在容器中运行。镜像的定制实际上就是定制每一层所添加的配置、文件。如果我们可以把每一层修改、安装、构建、操作的命令都写入一个脚本，用这个脚本来构建、定制镜像，十分利于批量定制镜像，也使镜像的内容变得透明。这个脚本就是 Dockerfile。

参考:

[菜鸟docker教程](https://www.runoob.com/docker/docker-hello-world.html)

[Docker Dockerfile 定制镜像](https://blog.csdn.net/wo18237095579/article/details/80540571)

[Dockerfile RUN,CMD,ENTRYPOINT命令三者区别](https://blog.csdn.net/liangweihua123/article/details/87436694)

[bash -c 注意事项](https://www.jianshu.com/p/198d819d24d1)

# 2 编写详解
Dockerfile 是一个文本文件，其内包含了一条条的指令(Instruction)，每一条指令构建一层，因此每一条指令的内容，就是描述该层应当如何构建。以定制 nginx 镜像为例，使用 Dockerfile 来定制。在一个空白目录中，建立一个文件（不要写后缀），并命名为 Dockerfile ：
```sh
$ mkdir mynginx
$ cd mynginx
$ nano Dockerfile
```
dockerfile内容为：
```dockerfile
FROM nginx
RUN echo '<h1>Hello, Docker!</h1>' > /usr/share/nginx/html/index.html
```
这个dockerfile的定制很简单，只包含了FROM和RUN两条指令，也就是在基础镜像 nginx 镜像 的基础上，多了一层定制。

接下来，开始逐条介绍常用的dockerfile指令。

# 2.1 FROM
