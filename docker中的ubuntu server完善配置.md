# 1 前言
在windows中安装 docker for windows 后，pull Ubuntu的镜像，在本地运行ubuntu 的container，以取代虚拟机。

记录一下，一步一步将docker里的Ubuntu镜像改造为合适正常开发的ubuntu server。

# 2 安装编辑器nano
 原本镜像甚至不自带编辑器，先安装编辑器，方便后面修改配置文件。
 ```bash
#  先启动ubuntu镜像
docker run -it ubuntu bash;

# 以root用户身份进入了ubunt的bash交互模拟终端
# 所以，ps -ef的第一个PID是 bash，如果该PID结束，就意味着该container的使命结束，
# 容器也会随着该进程的停止和停止！！！！

# 进入容器的交互命令终端后
apt update;
apt install nano;
```

对该镜像经过修改后，需要将这些修改持久化到镜像中，通过一下命令暂时退出模拟终端，但不会停止container的bash的进程：

```
ctrl + p
ctrl + q
```

然后，开始镜像的修改进行持久化：

```bash
# 先用命令查询当前容器的ID，假设查出来是 666666
docker ps;

# 将运行中的容器的文件状态持久化到镜像 my_ubuntu，也可以直接覆盖之前的镜像
docker commit 666666 my_ubuntu;

# 下次启动时就可以使用新镜像了
docker run -it my_ubuntu bash;
```

# 3 安装、配置ssh-server
```bash
# 安装openssh-server
apt install openssh-server;

# 查看当前的用户名
whoami;
# 必然是root
# docker默认只有root用户

# 修改当前用户的密码
passwd root;

# 一系列密码修改过程……

# openssh-server默认不允许使用的root用户登录
# 修改ssh server配置去除这一限制

# ssh是客户端，sshd的d是daemon，叫守护进程，其实就是windows里的service
# 本质就是后台进程，docker engine就是一个daemon
# 此处要修改的是守护进程的配置
nano /etc/ssh/sshd_config;

# 添加一行配置，允许root用户登录
PermitRootLogin yes

# 这时候可以启动sshd了，有两种方式：
service ssh start;
# 或者
/etc/init.d/ssh start
# 启动后，会使用默认的22端口


# 将sshd的自启动添加进系统
# 原先的Linux使用initd管理系统引导，较新的都直接使用systemd
# 在不使用initd的情况下，暂时没想到合适的方法，似乎只能编写dockeFile的文件，写进run.sh中了
```

linux 使用systemd设置自启动   https://blog.csdn.net/weixin_40700617/article/details/90780009

ubuntu18.04 initd和systemd介绍    https://www.cnblogs.com/qingshangucheng/p/10183583.html






