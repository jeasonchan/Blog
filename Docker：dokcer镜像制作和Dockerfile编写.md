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
```dockerfile
RUN echo '<h1>Hello, Docker!</h1>' > /usr/share/nginx/html/index.html
```
* exec 格式： RUN ["可执行文件", "参数1", "参数2"]，这更像是函数调用中的格式。比如：
```dockerfile
RUN ["/run.sh"]
```

因此，我们可以在一个Linux发行版镜像的基础上，通过RUN命令进行依赖的安装，比如：

```dockerfile
FROM debian:jessie
RUN apt-get update
RUN apt-get install -y gcc libc6-dev make
RUN wget -O redis.tar.gz "http://download.redis.io/releases/redis-3.2.5.tar.gz"
RUN mkdir -p /usr/src/redis
RUN tar -xzf redis.tar.gz -C /usr/src/redis --strip-components=1
RUN make -C /usr/src/redis
RUN make -C /usr/src/redis install
```

之前说过，Dockerfile 中每一个指令都会建立一层， RUN 也不例外。每一个 RUN 的行为，就和刚才我们手工建立镜像的过程一样：新建立一层，在其上执行这些命令，执行结束后， commit 这一层的修改，构成新的镜像。因此，7行的RUN指令会创建了 7 层镜像。这是完全没有意义的，而且很多运行时不需要的东西，都被装进了镜像里，比如编译环境、更新的软件包等等。结果就是产生非常臃肿、非常多层的镜像，不仅仅增加了构建部署的时间，也很容易出错。 这是很多初学 Docker 的人常犯的一个错误。

因此，上面的 Dockerfile 正确的写法应该是这样：
```dockerfile
FROM debian:jessie
RUN buildDeps='gcc libc6-dev make' \
    && apt-get update \
    && apt-get install -y $buildDeps \
    && wget -O redis.tar.gz "http://download.redis.io/releases/redis-3.2.5.tar.gz" \
    && mkdir -p /usr/src/redis \
    && tar -xzf redis.tar.gz -C /usr/src/redis --strip-components=1 \
    && make -C /usr/src/redis \
    && make -C /usr/src/redis install \
    && rm -rf /var/lib/apt/lists/* \
    && rm redis.tar.gz \
    && rm -r /usr/src/redis \
    && apt-get purge -y --auto-remove $buildDeps
```
对改进后的dockerfile文件进行解读：

首先，之前所有的命令只有一个目的，就是编译、安装 redis 可执行文件。因此没有必要建立很多层，这只是一层的事情。因此，这里没有使用很多个 RUN ，而是仅仅使用一个 RUN 指令，并使用 && 将各个所需命令串联起来。将之前的 7 层，简化为了1 层。在撰写 Dockerfile 的时候，要经常提醒自己，这并不是在写 Shell 脚本，而是在定义每一层该如何构建。

并且，这里为了格式化还进行了换行。Dockerfile 支持 Shell 命令的行尾添加 \ 的命令换行方式，以及行首 # 进行注释的格式。**良好的格式，比如换行、缩进、注释等，会让维护、排障更为容易，这是一个比较好的习惯。**

此外，还可以看到这一组命令的最后添加了清理工作的命令，删除了为了编译构建所需要的软件，清理了所有下载、展开的文件，并且还清理了 apt 缓存文件。这是很重要的一步，之前有说过，**镜像是多层存储，每一层的东西并不会在下一层被删除，会一直跟随着镜像。因此镜像构建时，一定要确保每一层只添加真正需要添加的东西，任何无关的东西都应该清理掉。**

很多人初学 Docker 制作出了很臃肿的镜像的原因之一，就是忘记了每一层构建的最后一定要清理掉无关文件。
## 2.3 COPY 复制文件
作用是，从**上下文（下文有详细介绍）**路径为起始的**相对路径**中拷贝文件，到容器的**绝对路径**中，使用格式为：
```dockerfile
COPY <源路径>... <目标路径>
COPY ["<源路径1>","<源路径2>"... "<目标路径>"]
```
如上所示，COPY 和 RUN 指令一样，也有两种格式，一种类似于命令行，一种类似于函数调用。COPY 指令将从构建上下文目录中 <源路径> 的文件/目录 复制到新的一层的镜像内的 <目标路径> 位置。比如：
```dockerfile
COPY package.json /usr/src/app/
```

<源路径> 可以是多个，甚至可以是通配符(和shell里的cp使用方式基本保持一致)，其通配符规则要满足 Go 的 filepath.Match 规则，如：
```dockerfile
COPY hom* /mydir/
COPY hom?.txt /mydir/
COPY 1.txt 2.txt /mydir/
```

<目标路径> 可以是容器内的绝对路径，也可以是相对于工作目录的相对路径（工作目录可以用 WORKDIR 指令来指定）。目标路径不需要事先创建，如果目录不存在会在复制文件前先行创建缺失目录。

此外，还需要注意一点，使用 COPY 指令，源文件的各种元数据都会保留。比如读、写、执行权限、文件变更时间等。这个特性对于镜像定制很有用。特别是构建相关文件都在使用 Git进行管理的时候。

## 2.4 ADD

## 2.5 CMD

# 3 构建镜像（docker builder）
docker builder命令，根据dockerfile进行镜像构建。



