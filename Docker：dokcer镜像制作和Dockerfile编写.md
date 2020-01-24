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
ADD 指令和 COPY 的格式和性质基本一致。但是在 COPY 基础上增加了一些功能。比如 <源路径> 可以是一个 URL 。

源路径时URL时，Docker 引擎会试图去下载这个链接的文件放到 <目标路径> 去。下载后的文件权限自动设置为 600 ，如果这并不是想要的权限，那么还需要增加额外的一层 RUN 进行权限调整，另外，如果下载的是个压缩包，需要解压缩，也一样还需要额外的一层 RUN 指令进行解压缩。所以不如直接使用 RUN 指令，然后使用 wget 或者 curl 工具下载，处理权限、解压缩、然后清理无用文件更合理。因此，这个功能其实并不实用，而且不推荐使用。

<源路径> 为一个 tar 压缩文件的话，压缩格式为 gzip , bzip2 以及 xz 的情况下， ADD 指令将会自动解压缩这个压缩文件到 <目标路径> 去。

在某些情况下，这个自动解压缩的功能非常有用，比如官方镜像 ubuntu 中：
```dockerfile
FROM scratch
ADD ubuntu-xenial-core-cloudimg-amd64-root.tar.gz /
```
但在某些情况下，如果我们真的是希望复制个压缩文件进去，而不解压缩，这时就不可以使用 ADD 命令了。

在 Docker 官方的 Dockerfile 最佳实践文档 中要求，尽可能的使用 COPY ，因为 COPY 的语义很明确，就是复制文件而已，而 ADD 则包含了更复杂的功能，其行为也不一定很清晰。最适合使用 ADD 的场合，就是所提及的需要自动解压缩的场合。

另外需要注意的是， ADD 指令会令镜像构建缓存失效，从而可能会令镜像构建变得比较缓慢。

因此在 COPY 和 ADD 指令中选择的时候，可以遵循这样的原则，所有的文件复制均使用 COPY 指令，仅在需要自动解压缩的场合使用 ADD 。

## 2.5 CMD
CMD的功能是，指定容器启动时的默认执行命令，即直接使用 docker run （这里没有指定任何命令） 镜像名 时，默认执行的命令。极为重要的是，CMD指定的指令的运行生命期，即为容器的运行状态的生命期。CMD指定的命令执行完（也就是运行生命期结束），docker stats查到的状态就是EXIT状态。

CMD 指令的格式和 RUN 相似，也是两种格式：
* shell 格式： CMD <命令>
* exec 格式： CMD ["可执行文件", "参数1", "参数2"...]
* 特殊用法，只当作参数列表。 CMD ["参数1", "参数2"...] 。只要指定了 ENTRYPOINT 指令，用 CMD 只能用于指定具体的参数。

Docker 不是虚拟机，容器就是进程。既然是进程，那么在启动容器的时候，需要指定所运行的程序及参数。 CMD 指令就是用于指定默认的容器主进程的启动命令的。

在运行时可以指定新的命令来替代镜像设置中的这个默认命令，比如， ubuntu 镜像默认的CMD 是 /bin/bash ，如果我们直接 docker run -it ubuntu 的话，会直接进入 bash 。我们也可以在运行时指定运行别的命令，如 docker run -it ubuntu cat /etc/os-release 。这就是用 cat /etc/os-release 命令替换了默认的 /bin/bash 命令了，输出了系统版本信息。

在指令格式上，一般推荐使用 exec 格式，这类格式在解析时会被解析为 JSON 数组，因此**一定要使用双引号"xxxx "**，而不要使用单引号。

如果使用 shell 格式的话，实际的命令会被包装为 sh -c 的参数的形式进行执行。比如：
```dockerfile
CMD echo $HOME
```
在实际执行中，会将其变更为：
```dockerfile
CMD [ "sh", "-c", "echo $HOME" ]
# 也就是执行  sh -c "echo $HOME"
```
这就是为什么我们可以使用环境变量的原因，因为这些环境变量会被 shell 进行解析处理。提到 CMD 就不得不提容器中应用在前台执行和后台执行的问题。这是初学者常出现的一个混淆。

Docker 不是虚拟机，容器中的应用都应该以前台执行，而不是像虚拟机、物理机里面那样，用 upstart/systemd 去启动后台服务，容器内没有后台服务的概念。比如，初学者一般将 CMD 写为：
```dockerfile
CMD service nginx start
```
然后发现容器执行后就立即退出了。甚至在容器内去使用 systemctl 命令结果却发现根本执行不了。这就是因为没有搞明白前台、后台的概念，没有区分容器和虚拟机的差异，依旧在以传统虚拟机的角度去理解容器。对于容器而言，其启动程序就是容器应用进程，容器就是为了主进程而存在的，主进程退出，容器就失去了存在的意义，从而退出，其它辅助进程不是它需要关心的东西。

而使用 service nginx start 命令，则是希望 systemd 来以后台守护进程形式启动 nginx 服务。而刚才说了 CMD service nginx start 会被理解为 CMD [ “sh”, “-c”, “service nginxstart”] ，因此主进程实际上是 sh 。那么当 service nginx start 命令结束后， sh 也就结束了， sh 作为主进程退出了，自然就会令容器退出。

正确的做法是直接执行 nginx 可执行文件，并且要求以前台形式运行。比如：
```dockerfile
CMD ["nginx", "-g", "daemon off;"]
```
## 2.6 ENTRYPOINT
ENTRYPOINT 的目的和 CMD 一样，都是在指定容器启动程序及参数。ENTRYPOINT 在运行时也可以替代，不过比 CMD 要略显繁琐，需要通过 docker run 的参数 –entrypoint 来指定。

ENTRYPOINT 的格式和 RUN 指令格式一样，分为 exec 格式和 shell 格式。

当指定了 ENTRYPOINT 后， CMD 的含义就发生了改变，不再是直接的运行其命令，而是将CMD 的内容作为参数传给 ENTRYPOINT 指令，换句话说实际执行时，将变为：
```sh
<ENTRYPOINT指定的指定文件> "<CMD中的参数>"
```
那么有了 CMD 后，为什么还要有 ENTRYPOINT 呢？这种 <ENTRYPOINT> "<CMD>" 有什么好处么？让我们来看两个场景。
### 场景一：让镜像变成像命令一样使用
假设我们需要一个得知自己当前公网 IP 的镜像，那么可以先用 CMD 来实现：
```dockerfile
FROM ubuntu:16.04
RUN apt-get update \
&& apt-get install -y curl \
&& rm -rf /var/lib/apt/lists/*
CMD [ "curl", "-s", "http://ip.cn" ]
```
假如我们使用 docker build -t myip . 来构建镜像的话，如果我们需要查询当前公网 IP，只需要执行：
```sh
$ docker run myip
当前 IP：61.148.226.66 来自：北京市 联通
```
嗯，这么看起来好像可以直接把镜像当做命令使用了，不过命令总有参数，如果我们希望加参数呢？比如从上面的 CMD 中可以看到实质的命令是 curl ，那么如果我们希望显示 HTTP头信息，就需要加上 -i 参数。那么我们可以直接加 -i 参数给 docker run myip 么？
```sh
$ docker run myip -i
docker: Error response from daemon: invalid header field value "oci runtime error: con
tainer_linux.go:247: starting container process caused \"exec: \\\"-i\\\": executable
file not found in $PATH\"\n".
```
我们可以看到可执行文件找不到的报错， executable file not found 。之前我们说过，跟在镜像名后面的是 command ，运行时会替换 CMD 的默认值。因此这里的 -i 替换了原来的 CMD ，而不是添加在原来的 curl -s http://ip.cn 后面。而 -i 根本不是命令，所以自然找不到。

那么如果我们希望加入 -i 这参数，我们就必须重新完整的输入这个命令：
```sh
$ docker run myip curl -s http://ip.cn -i
```
这显然不是很好的解决方案，而使用 ENTRYPOINT 就可以解决这个问题。现在我们重新用 ENTRYPOINT 来实现这个镜像：
```dockerfile
FROM ubuntu:16.04
RUN apt-get update \
    && apt-get install -y curl \
    && rm -rf /var/lib/apt/lists/*
ENTRYPOINT [ "curl", "-s", "http://ip.cn" ]
```
这次我们再来尝试直接使用 docker run myip -i ：
```sh
$ docker run myip
当前 IP：61.148.226.66 来自：北京市 联通
$ docker run myip -i
HTTP/1.1 200 OK
Server: nginx/1.8.0
Date: Tue, 22 Nov 2016 05:12:40 GMT
Content-Type: text/html; charset=UTF-8
Vary: Accept-Encoding
X-Powered-By: PHP/5.6.24-1~dotdeb+7.1
X-Cache: MISS from cache-2
X-Cache-Lookup: MISS from cache-2:80
X-Cache: MISS from proxy-2_6
Transfer-Encoding: chunked
Via: 1.1 cache-2:80, 1.1 proxy-2_6:8006
Connection: keep-alive

当前 IP：61.148.226.66 来自：北京市 联通
```
可以看到，这次成功了。这是因为当存在 ENTRYPOINT 后， CMD 的内容将会作为参数传给 ENTRYPOINT ，而这里 -i 就是新的 CMD ，因此会作为参数传给 curl ，从而达到了我们预期的效果。

### 场景二：应用运行前的准备工作
启动容器就是启动主进程，但有些时候，启动主进程前，需要一些准备工作。比如 mysql 类的数据库，可能需要一些数据库配置、初始化的工作，这些工作要在最终的 mysql 服务器运行之前解决。

此外，可能希望避免使用 root 用户去启动服务，从而提高安全性，而在启动服务前还需要以 root 身份执行一些必要的准备工作，最后切换到服务用户身份启动服务。或者除了服务外，其它命令依旧可以使用 root 身份执行，方便调试等。

这些准备工作是和容器 CMD 无关的，无论 CMD 为什么，都需要事先进行一个预处理的工作。这种情况下，可以写一个脚本，然后放入 ENTRYPOINT 中去执行，而这个脚本会将接到的参数（也就是 ）作为命令，在脚本最后执行。比如官方镜像 redis 中就是这么做的：
```dockerfile
FROM alpine:3.4
RUN addgroup -S redis && adduser -S -G redis redis
ENTRYPOINT ["docker-entrypoint.sh"]
EXPOSE 6379
CMD [ "redis-server" ]
```
可以看到其中为了 redis 服务创建了 redis 用户，并在最后指定了 ENTRYPOINT 为 dockerentrypoint.sh 脚本，脚本内容是：
```sh
#!/bin/sh
# allow the container to be started with `--user`
if [ "$1" = 'redis-server' -a "$(id -u)" = '0' ]; then
    chown -R redis .
    exec su-exec redis "$0" "$@"
fi
exec "$@"
```
该脚本的内容就是根据 CMD 的内容来判断，如果是 redis-server 的话，则切换到 redis 用户身份启动服务器，否则依旧使用 root 身份执行。比如：
```sh
$ docker run -it redis id
uid=0(root) gid=0(root) groups=0(root)
```
## 2.6 ENV
这个指令很简单，就是设置环境变量而已，无论是后面的其它指令，如 RUN ，还是运行时的应用，都可以直接使用这里定义的环境变量。格式有两种：
```dockerfile
ENV <key> <value>
ENV <key1>=<value1> <key2>=<value2>...
```
比如：
```dockerfile
ENV VERSION=1.0 DEBUG=on \
    NAME="Happy Feet"
```
这个例子中演示了如何换行，以及对含有空格的值用双引号括起来的办法，这和 Shell 下的行为是一致的。

定义了环境变量，那么在后续的指令中，就可以使用这个环境变量。比如在官方 node 镜像 Dockerfile 中，就有类似这样的代码：
```dockerfile
ENV NODE_VERSION 7.2.0
RUN curl -SLO "https://nodejs.org/dist/v$NODE_VERSION/node-v$NODE_VERSION-linux-x64.ta
r.xz" \
    && curl -SLO "https://nodejs.org/dist/v$NODE_VERSION/SHASUMS256.txt.asc" \
    && gpg --batch --decrypt --output SHASUMS256.txt SHASUMS256.txt.asc \
    && grep " node-v$NODE_VERSION-linux-x64.tar.xz\$" SHASUMS256.txt | sha256sum -c - \
    && tar -xJf "node-v$NODE_VERSION-linux-x64.tar.xz" -C /usr/local --strip-components=1 \
    && rm "node-v$NODE_VERSION-linux-x64.tar.xz" SHASUMS256.txt.asc SHASUMS256.txt \
    && ln -s /usr/local/bin/node /usr/local/bin/nodejs
```
在这里先定义了环境变量 NODE_VERSION ，其后的 RUN 这层里，多次使用 $NODE_VERSION 来进行操作定制。可以看到，将来升级镜像构建版本的时候，只需要更新 7.2.0 即可， Dockerfile 构建维护变得更轻松了。

下列指令可以支持环境变量展开：
  ADD 、 COPY 、 ENV 、 EXPOSE 、 LABEL 、 USER 、 WORKDIR 、 VOLUME 、 STOPSIGNAL 、 ONBUILD 。

可以从这个指令列表里感觉到，环境变量可以使用的地方很多，很强大。通过环境变量，我们可以让一份 Dockerfile 制作更多的镜像，只需使用不同的环境变量即可。

## 2.7 ARG 构建参数
构建参数和 ENV 的效果一样，都是设置环境变量。所不同的是， ARG 所设置的构建环境的环境变量，在将来容器运行时是不会存在这些环境变量的。但是不要因此就使用 ARG 保存密码之类的信息，因为 docker history 还是可以看到所有值的。格式： 
```dockerfile
ARG <参数名>[=<默认值>]
```
Dockerfile 中的 ARG 指令是定义参数名称，以及定义其默认值。该默认值可以在构建命令 docker build 中用 --build-arg <参数名>=<值> 来覆盖，比如：
```sh
docker build . ubuntu --build-arg hahaha=23333
```
在 1.13 之前的版本，要求 –build-arg 中的参数名，必须在 Dockerfile 中用 ARG 定义过了，换句话说，就是 –build-arg 指定的参数，必须在 Dockerfile 中使用了。如果对应参数没有被使用，则会报错退出构建。从 1.13 开始，这种严格的限制被放开，不再报错退出，而是显示警告信息，并继续构建。这对于使用 CI 系统，用同样的构建流程构建不同的 Dockerfile 的时候比较有帮助，避免构建命令必须根据每个 Dockerfile 的内容修改。

## 2.8 VOLUME 定义持久化的匿名卷
容器运行时应该尽量保持容器存储层不发生写操作，对于数据库类需要保存动态数据的应用，其数据库文件应该保存于卷(volume)中。VOLUME就是使容器中的一个目录具有持久化存储数据的功能，该目录可以被容器本身使用，也可以共享给其他容器使用（VOLUME指定同一路径即可。）。持久化能力，本质上还是借助存储在宿主机的匿名文件/文件夹来实现的。dockerfile中的格式为：
```dockerfile
VOLUME ["<路径1>", "<路径2>"...]
VOLUME <路径>
```
比如：
```dockerfile
VOLUME /data
# 每次容器运行期对该目录所做的修改都会保存下来
```
这里的 /data 目录就会在运行时自动挂载为产生匿名文件/文件夹，任何向 /data 中写入的信息都不会记录进容器存储层，而是写进宿主机的文件/文件夹中，从而保证了容器存储层的无状态化。当然，运行时可以覆盖这个挂载设置。比如：
```sh
docker run -d -v mydata:/data xxxx
```
在这行命令中，就使用了 mydata 这个命名卷挂载到了 /data 这个位置，替代了 Dockerfile 中定义的匿名卷的挂载配置。

## 2.9 EXPOSE 声明端口
EXPOSE 指令是声明运行时容器提供服务端口，这**只是一个声明，在运行时并不会因为这个声明应用就会开启这个端口的服务**。在 Dockerfile 中写入这样的声明有两个好处，一个是帮助镜像使用者理解这个镜像服务的守护端口，以方便配置映射；另一个用处则是在运行时使用随机端口映射时，也就是 docker run -P 时，会自动随机映射 EXPOSE 的端口。格式为:
```dockerfile
EXPOSE <端口1> [<端口2>...]。
```

注意！！！！！！要将 EXPOSE 和在运行时使用 -p <宿主端口>:<容器端口> 区分开来。 -p ，是映射宿主端口和容器端口，换句话说，就是将容器的对应端口服务公开给外界访问，而 EXPOSE 仅仅是声明容器打算使用什么端口而已，并不会自动在宿主进行端口映射。

## 2.10 WORKDIR 指定工作目录
使用 WORKDIR 指令可以来指定工作目录（或者称为当前目录），以后各层的当前目录就被改为指定的目录，如该目录不存在， WORKDIR 会帮你建立目录。格式为:
```dockerfile
WORKDIR <工作目录路径> 。
```
之前提到一些初学者常犯的错误是把 Dockerfile 等同于 Shell 脚本来书写，这种错误的理解还可能会导致出现下面这样的错误：

RUN cd /app
RUN echo "hello" > world.txt
1
2
  如果将这个 Dockerfile 进行构建镜像运行后，会发现找不到 /app/world.txt 文件，或者其内容不是 hello 。原因其实很简单，在 Shell 中，连续两行是同一个进程执行环境，因此前一个命令修改的内存状态，会直接影响后一个命令；而在 Dockerfile 中，这两行 RUN 命令的执行环境根本不同，是两个完全不同的容器。这就是对 Dockerfile 构建分层存储的概念不了解所导致的错误。

  之前说过每一个 RUN 都是启动一个容器、执行命令、然后提交存储层文件变更。第一层 RUNcd /app 的执行仅仅是当前进程的工作目录变更，一个内存上的变化而已，其结果不会造成任何文件变更。而到第二层的时候，启动的是一个全新的容器，跟第一层的容器更完全没关系，自然不可能继承前一层构建过程中的内存变化。

  因此如果需要改变以后各层的工作目录的位置，那么应该使用 WORKDIR 指令。

## 2.11 USER 指定当前用户

## 2.12 HEALTHCHECK 健康检查

# 3 构建镜像（docker builder）
docker builder命令，根据dockerfile进行镜像构建。



