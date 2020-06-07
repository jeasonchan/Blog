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
