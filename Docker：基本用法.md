# 1 背景
买了阿里云，发现用dockers部署应用十分方便，尤其不会污染宿主机！十分便利！之前的部署都是跟着教程来的，没有系统的学习相关知识，现在系统学习一下基本的东西。

参考<<https://www.runoob.com/docker/docker-hello-world.html>>。

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

* **运行交互式容器**

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



