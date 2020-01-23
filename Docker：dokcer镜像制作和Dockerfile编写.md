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

## 2.1 FROM
所谓定制镜像，那一定是以一个镜像为基础，在其上进行定制。而 FROM 就是指定基础镜像，因此一个 Dockerfile 中 FROM 是必备的指令，并且必须是第一条指令。

在 Docker Store 上有非常多的高质量的官方镜像，有可以直接拿来使用的服务类的镜像，如nginx 、 redis 、 mongo 、mysql 等；也有一些方便开发、构建、运行各种语言应用的镜像，如 node 、 openjdk 、 python 等。可以在其中寻找一个最符合我们最终目标的镜像为基础镜像进行定制。

如果没有找到对应服务的镜像，官方镜像中还提供了一些更为基础的操作系统镜像，如ubuntu 、 debian 、 centos 等，这些操作系统的软件库为我们提供了更广阔的扩展空间。

## 2.2 RUN
RUN 指令是用来执行命令行命令的。由于命令行的强大能力， RUN 指令在定制镜像时是最常用的指令之一。其格式有两种：

* shell 格式： RUN <命令> ，就像直接在命令行中输入的命令一样。刚才写的 Dockerfile 中的 RUN 指令就是这种格式，比如：
```sh
RUN echo '<h1>Hello, Docker!</h1>' > /usr/share/nginx/html/index.html
```
* exec 格式： RUN ["可执行文件", "参数1", "参数2"]，这更像是函数调用中的格式。比如：
```sh
RUN ["./run.sh"]
```

既然 RUN 就像 Shell 脚本一样可以执行命令，那么我们是否就可以像 Shell 脚本一样把每个命令对应一个 RUN 呢？比如这样：

```sh
FROM debian:jessie
RUN apt-get update
RUN apt-get install -y gcc libc6-dev make
RUN wget -O redis.tar.gz "http://download.redis.io/releases/redis-3.2.5.tar.gz"
RUN mkdir -p /usr/src/redis
RUN tar -xzf redis.tar.gz -C /usr/src/redis --strip-components=1
RUN make -C /usr/src/redis
RUN make -C /usr/src/redis install
```

# 3 docker builder
docker builder命令，使用dockerfile进行镜像构建。



