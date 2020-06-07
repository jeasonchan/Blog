# 1 前言
之前使用mysql都是直接安装mysql应用，然后在本地使用mysqld命令启动起来，其余的中间件也是采用类似的方式，这样就带来的一些问题，使用的物理机的PATH中的增加了很多bin的路径，而且启动的参数也各式各样的，于是想到在ubuntu中使用docker来运行一些中间件实例。

接下来从0开始讲解，在ubuntu中使用docker配配置、运行mysql实例。

# 2 安装、配置docker

## 2.1 apt install
直接使用：

```bash
sudo apt install dokcer.io
```

到现在还有很多教程在放屁，让安装docker ce……真的副了，人云亦云……很久之前，docker.io 由ubuntu团队维护，暂停维护了一段时间后，目前已经恢复正常维护了，且docker的版本和官方的docker ce版本基本一致，所以，现在建议直接在ubuntu中使用docker.io。

## 2.2  配置面免密使用docker命令
要使用的docker命令，一般都是必须使用root权限才能运行起来，在生产环境中无可非议，这样做能保证服务的安全，但是，自己的平时练习就么有必要了，因为：

1. 每次都要输入sudo很麻烦；如果，su root，docker会缺少命令补全功能……
2. vscode是以普通用户的身份启动的，里面的插件docker访问本机运行的docker当要也需要root权限，很显然以普通用户身份启动的vscode无法正常使用需要root权限的docker

解决方法就是，**将当前用户加入到docker用户组中**。

```bash
# 创建用户组 docker
sudo groupadd docker

# 打印一下当前的用户名称
echo $USER

# 经当前用户加入到docker组中
sudo usermod -aG docker $USER
```

重启系统即可，就能以普通用户的身份运行docker命令

## 2.3 配置自启动
众所周知，docker其实是一个b/s结构的本地应用。server端是运行在本地的，windows中叫服务（service），linux中叫守护进程（daemon），linux中手动启动一个守护进程的方式可以用：

```bash
# 启动
service docker start
# 停止
service docker stop
```

没有开机就启动的需求，每次要用的时候手动启动即可，要想开机启动可以通过systemctl命令实现开机对该程序的引导执行。

```bash
# 
$ sudo systemctl enable docker

# To disable this behavior, use disable instead.

$ sudo systemctl disable docker
```

## 2.4 优化镜像下载速度
经过之前的一些配置，守护进程已经启动的情况下，vscode的docker插件也应该是能正常使用了。

但是！使用dokcer pull去下载dockerHub中的公共镜像的时候，dockerHub默认的服务器在国外，下载会比较慢，建议切换成国内的云服务商提供的加速镜像地址。

1. 注册并登陆阿里云账号
2. 进入 https://cr.console.aliyun.com/cn-hangzhou/instances/repositories  阿里的容器镜像服务地址
3. 左下角"镜像加速"，已经自动生成了切换镜像的命令行：

```bash
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://每个人都不一样的字符串.mirror.aliyuncs.com"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```

本质上就是创建了一个配置文件：/etc/docker/daemon.json，向里面写入了镜像地址，然后通过systemctl命令重启了守护进程


经过一番配置，下面就可以开始下载mysql镜像，并运行了

# 3 拉取、运行、配置mysql镜像

```bash
# latest可以省略
docker pull mysql:latest

# 罗列一下本地的镜像
docker images

REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
redis               latest              36304d3b4540        9 days ago          104MB
mysql               latest              30f937e841c8        2 weeks ago         541MB

# 先尝试运行刚刚拉取的镜像，可以通过镜像名，也可以通过镜像ID
docker run 30f937e841c8
2020-06-07 15:03:22+00:00 [Note] [Entrypoint]: Entrypoint script for MySQL Server 8.0.20-1debian10 started.
2020-06-07 15:03:23+00:00 [Note] [Entrypoint]: Switching to dedicated user 'mysql'
2020-06-07 15:03:23+00:00 [Note] [Entrypoint]: Entrypoint script for MySQL Server 8.0.20-1debian10 started.
2020-06-07 15:03:23+00:00 [ERROR] [Entrypoint]: Database is uninitialized and password option is not specified
	You need to specify one of MYSQL_ROOT_PASSWORD, MYSQL_ALLOW_EMPTY_PASSWORD and MYSQL_RANDOM_ROOT_PASSWORD
#报错了，每运行起来，docker ps也查不到其正在运行， 需要我们至少指明三个密码其中的一个

# -d 以后台方式运行，而不再占用当前的终端
# -p 容器内的端口映射到外部
# --name 指明实例名称
# -e 设置容器运行用到的环境变量
# 最后一个mysql是本地的镜像名称
docker run -d -p 3306:3306 --name mymysql -e MYSQL_ROOT_PASSWORD=123456 mysql

# 查看一下运行状态
docker stats

CONTAINER ID        NAME                CPU %               MEM USAGE / LIMIT     MEM %               NET I/O             BLOCK I/O           PIDS
daf16e32d468        mymysql             0.75%               324.5MiB / 11.39GiB   2.78%               3.86kB / 0B         37MB / 471MB        38

# 使用DataGrip连接一下，已能正常使用

# 将当前可以用的mysql实例保存一下，供以后使用
docker commit daf16e32d468 mysql:latest_1

# 查看一下刚刚提交的镜像
docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
mysql               latest_1            a314d06a41a5        3 minutes ago       541MB
redis               latest              36304d3b4540        9 days ago          104MB
mysql               latest              30f937e841c8        2 weeks ago         541MB

# 尝试运行一下
docker run -p 3306:3306   mysql:latest_1
2020-06-07 15:32:05+00:00 [Note] [Entrypoint]: Entrypoint script for MySQL Server 8.0.20-1debian10 started.
2020-06-07 15:32:05+00:00 [Note] [Entrypoint]: Switching to dedicated user 'mysql'
2020-06-07 15:32:05+00:00 [Note] [Entrypoint]: Entrypoint script for MySQL Server 8.0.20-1debian10 started.
2020-06-07 15:32:06+00:00 [Note] [Entrypoint]: Initializing database files
2020-06-07T15:32:06.024984Z 0 [Warning] [MY-011070] [Server] 'Disabling symbolic links using --skip-symbolic-links (or equivalent) is the default. Consider not using this option as it' is deprecated and will be removed in a future release.
2020-06-07T15:32:06.025133Z 0 [System] [MY-013169] [Server] /usr/sbin/mysqld (mysqld 8.0.20) initializing of server in progress as process 42
2020-06-07T15:32:06.031952Z 1 [System] [MY-013576] [InnoDB] InnoDB initialization has started.
2020-06-07T15:32:07.225114Z 1 [System] [MY-013577] [InnoDB] InnoDB initialization has ended.
2020-06-07T15:32:09.890345Z 6 [Warning] [MY-010453] [Server] root@localhost is created with an empty password ! Please consider switching off the --initialize-insecure option.
2020-06-07 15:32:15+00:00 [Note] [Entrypoint]: Database files initialized
2020-06-07 15:32:15+00:00 [Note] [Entrypoint]: Starting temporary server
2020-06-07T15:32:15.370503Z 0 [Warning] [MY-011070] [Server] 'Disabling symbolic links using --skip-symbolic-links (or equivalent) is the default. Consider not using this option as it' is deprecated and will be removed in a future release.
2020-06-07T15:32:15.370653Z 0 [System] [MY-010116] [Server] /usr/sbin/mysqld (mysqld 8.0.20) starting as process 89
2020-06-07T15:32:15.397683Z 1 [System] [MY-013576] [InnoDB] InnoDB initialization has started.
2020-06-07T15:32:15.683794Z 1 [System] [MY-013577] [InnoDB] InnoDB initialization has ended.
2020-06-07T15:32:15.801544Z 0 [System] [MY-011323] [Server] X Plugin ready for connections. Socket: '/var/run/mysqld/mysqlx.sock'
2020-06-07T15:32:16.047661Z 0 [Warning] [MY-010068] [Server] CA certificate ca.pem is self signed.
2020-06-07T15:32:16.052051Z 0 [Warning] [MY-011810] [Server] Insecure configuration for --pid-file: Location '/var/run/mysqld' in the path is accessible to all OS users. Consider choosing a different directory.
2020-06-07T15:32:16.085802Z 0 [System] [MY-010931] [Server] /usr/sbin/mysqld: ready for connections. Version: '8.0.20'  socket: '/var/run/mysqld/mysqld.sock'  port: 0  MySQL Community Server - GPL.
2020-06-07 15:32:16+00:00 [Note] [Entrypoint]: Temporary server started.
Warning: Unable to load '/usr/share/zoneinfo/iso3166.tab' as time zone. Skipping it.
Warning: Unable to load '/usr/share/zoneinfo/leap-seconds.list' as time zone. Skipping it.
Warning: Unable to load '/usr/share/zoneinfo/zone.tab' as time zone. Skipping it.
Warning: Unable to load '/usr/share/zoneinfo/zone1970.tab' as time zone. Skipping it.

2020-06-07 15:32:18+00:00 [Note] [Entrypoint]: Stopping temporary server
2020-06-07T15:32:19.006840Z 10 [System] [MY-013172] [Server] Received SHUTDOWN from user root. Shutting down mysqld (Version: 8.0.20).
2020-06-07T15:32:21.738005Z 0 [System] [MY-010910] [Server] /usr/sbin/mysqld: Shutdown complete (mysqld 8.0.20)  MySQL Community Server - GPL.
2020-06-07 15:32:22+00:00 [Note] [Entrypoint]: Temporary server stopped

2020-06-07 15:32:22+00:00 [Note] [Entrypoint]: MySQL init process done. Ready for start up.

2020-06-07T15:32:22.262917Z 0 [Warning] [MY-011070] [Server] 'Disabling symbolic links using --skip-symbolic-links (or equivalent) is the default. Consider not using this option as it' is deprecated and will be removed in a future release.
2020-06-07T15:32:22.263060Z 0 [System] [MY-010116] [Server] /usr/sbin/mysqld (mysqld 8.0.20) starting as process 1
2020-06-07T15:32:22.274919Z 1 [System] [MY-013576] [InnoDB] InnoDB initialization has started.
2020-06-07T15:32:22.561633Z 1 [System] [MY-013577] [InnoDB] InnoDB initialization has ended.
2020-06-07T15:32:22.689342Z 0 [System] [MY-011323] [Server] X Plugin ready for connections. Socket: '/var/run/mysqld/mysqlx.sock' bind-address: '::' port: 33060
2020-06-07T15:32:22.824900Z 0 [Warning] [MY-010068] [Server] CA certificate ca.pem is self signed.
2020-06-07T15:32:22.829876Z 0 [Warning] [MY-011810] [Server] Insecure configuration for --pid-file: Location '/var/run/mysqld' in the path is accessible to all OS users. Consider choosing a different directory.
2020-06-07T15:32:22.858027Z 0 [System] [MY-010931] [Server] /usr/sbin/mysqld: ready for connections. Version: '8.0.20'  socket: '/var/run/mysqld/mysqld.sock'  port: 3306  MySQL Community Server - GPL.


# 已能成功运行，DataGrip也能正常访问数据，可见root用户的密码已经持久化到镜像中

```

可见，容器内的mysql运行时是可以直接从系统环境变量中取值的，个人推测应该是mysql的启动脚本做了一写检查判断。再结合之前的mysql使用经验，想改变容器内的nysql的实例参数，至少有以下的方法：

1. 设置容器的环境变量
2. 直接编写容器内的配置文件

# 3.1 编写容器内配置文件优化内存占用