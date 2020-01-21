# 1 背景
买了阿里云，发现用dockers部署应用十分方便，尤其不会污染宿主机！十分便利！之前的部署都是跟着教程来的，没有系统的学习相关知识，现在系统学习一下基本的东西。

参考:

[菜鸟docker教程](https://www.runoob.com/docker/docker-hello-world.html)

[Docker Dockerfile 定制镜像](https://blog.csdn.net/wo18237095579/article/details/80540571)

[Dockerfile RUN,CMD,ENTRYPOINT命令三者区别](https://blog.csdn.net/liangweihua123/article/details/87436694)

[bash -c 注意事项](https://www.jianshu.com/p/198d819d24d1)


# 2 安装
Windows上使用docker体验不好，完全没有Linux上的便利感……建议不如直接安装VMware然后安装Ubuntu server使用，安装命令很简单：
```sh
sudo apt install docker.io 
```
因为，在Ubuntu的仓库里，已经有其他的不相关的软件已经使用了docker这个软件名，所有，真正的docker安装名称是docker.io。

# 3 使用
# 3.1 基本范式
* **docker run，直接在容器中执行命令**，会在当前控制台输出，就行没有nohup一样。

Docker 可以让我们在容器内运行应用程序， 使用 docker run 命令来在容器内运行一个应用程序，比如：
```sh
jeason@ubuntu:~$ docker run ubuntu:15.10 /bin/echo "Hello world"
Hello world
```
对各个参数解析：

docker: Docker 的二进制执行文件。

run: 与前面的 docker 组合来运行一个容器。

ubuntu:15.10 指定要运行的镜像，Docker 首先从本地主机上查找镜像是否存在，如果不存在，Docker 就会从镜像仓库 Docker Hub 下载公共镜像。

/bin/echo "Hello world": 在启动的容器里执行的命令。在Ubuntu镜像里，/bin/echo可以简写为echo，正剧即可简写为echo "Hello world"。

以上命令完整的意思可以解释为：Docker 以 ubuntu15.10 镜像**创建一个新容器**，然后在容器里执行 bin/echo "Hello world"，然后输出结果。

---

* **启动一个镜像，并运行交互式容器**

我们通过 docker 的两个参数 -i -t，让 docker 运行的容器实现"对话"的能力：
```sh
jeason@ubuntu:~$ docker run -i -t ubuntu:15.10 /bin/bash
# 可以简写为 docker run -it ubuntu:15.10 bash
root@0123ce188bd8:/#
```
各个参数解析：

-t: 在新容器内指定一个伪终端或终端，也就是将容器内的终端转发到我们这一层来

-i: 允许你对容器内的标准输入 (STDIN) 进行交互。

通过-i -t 这两个参数，注意一下第二行 root@0123ce188bd8:/#，此时我们已**进入（其实，只是转发）**一个 ubuntu15.10 系统的容器，尝试在容器中运行命令 cat /proc/version和ls分别查看当前系统的版本信息和当前目录下的文件列表
```sh
root@0123ce188bd8:/#  cat /proc/version
Linux version 4.4.0-151-generic (buildd@lgw01-amd64-043) (gcc version 5.4.0 20160609 (Ubuntu 5.4.0-6ubuntu1~16.04.10) ) #178-Ubuntu SMP Tue Jun 11 08:30:22 UTC 2019
root@0123ce188bd8:/# ls
bin  boot  dev  etc  home  lib  lib64  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
root@0123ce188bd8:/# 
```
我们可以通过运行 exit 命令或者使用 CTRL+D 来退出容器。
```sh
root@0123ce188bd8:/#  exit
exit
jeason@ubuntu:~# 
``
注意第三行中 jeason@ubuntu 表明我已经退出了当期的容器，返回到当前的主机中。

* **容器后台运行程序**

```sh
jeason@ubuntu:~$ docker run -d ubuntu:15.10 /bin/sh -c "while true; do echo hello world; sleep 1; done"
2b1b7a428627c51ab8810d541d759f072b4fc75487eed05812646b8534a2fe63
```
参数解析：

docker run：容器运行命令

-d 参数：后台运行容器，并返回容器ID；和第一种的区别就体现出来了。

ubuntu:15.10：运行的目标镜像

在输出中，我们没有看到期望的 "hello world"，而是一串长字符：2b1b7a428627c51ab8810d541d759f072b4fc75487eed05812646b8534a2fe63，这个长字符串叫做**容器 ID**，对每个容器来说都是唯一的，我们可以通过容器 ID 来查看对应的容器发生了什么，或者说，查看容器的状态

首先，我们需要确认容器有在运行，可以通过 docker ps 来查看：
```sh
runoob@runoob:~$ docker ps
CONTAINER ID        IMAGE                  COMMAND              ...  
5917eac21c36        ubuntu:15.10           "/bin/sh -c 'while t…"    ...
```
输出的详情共有5列：
* CONTAINER ID: 容器 ID。
* IMAGE: 使用的镜像。
* COMMAND: 启动容器时运行的命令，也就是 docker run ………… command args中的，command args
* CREATED: 容器的创建时间。
* STATUS: 容器状态，状态有7种：created（已创建），restarting（重启中），running（运行中），removing（迁移中），paused（暂停），exited（停止），dead（死亡）
* PORTS: 容器的端口信息和使用的连接类型（tcp\udp）。
* NAMES: 自动分配的容器名称。


---

* **让一个运行中的镜像执行命令**
```sh
docker exec -it containerID sh
```
参数解析：

exec：和run唯一区别是，run是从一个镜像启动容器实例，exec是直接选取一个存在的容器实例

---
* **剩下的用法较为简单，可直接使用docker --help进行查看**
