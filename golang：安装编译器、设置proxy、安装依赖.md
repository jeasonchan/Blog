# 1 背景
最近要用golang写插件，整理记录一下golang编程环境搭建的工作


# 2 安装编译器
在linux进行开发工作，发行版用的是rpm包管理器。

```bash
sudo yum search golang

# 查看一下库包信息，因为插件是基于模块系统的，1.11之后的才支持模块的特性
sudo yum info golang.x86_64

sudo yum install golang.x86_64
```

安装后涉及的目录：

可执行文件位置：  /usr/bin/go

源码及安装的依赖的目录位置:  /usr/lib/golang/,该目录下面的api/ bin/ lib/ pkg/ src/，之后离线安装插件会使用到

# 3 设置代理
golang官方支持类似maven、pip一样的依赖库，比如，在中国的镜像/代理就叫https://goproxy.cn，这一步将其设置为代理:

```bash
# 先查看一下go相关的环境变量值
go env

# 输出如下
GOARCH="amd64"
GOBIN=""
GOCACHE="/home/jeasonchan/.cache/go-build"
GOEXE=""
GOFLAGS=""
GOHOSTARCH="amd64"
GOHOSTOS="linux"
GOOS="linux"
GOPATH="/home/jeasonchan/go"
GOPROXY=""
GORACE=""
GOROOT="/usr/lib/golang"
GOTMPDIR=""
GOTOOLDIR="/usr/lib/golang/pkg/tool/linux_amd64"
GCCGO="gccgo"
CC="gcc"
CXX="g++"
CGO_ENABLED="1"
GOMOD="/home/jeasonchan/projects/monstache-rel6/go.mod"
CGO_CFLAGS="-g -O2"
CGO_CPPFLAGS=""
CGO_CXXFLAGS="-g -O2"
CGO_FFLAGS="-g -O2"
CGO_LDFLAGS="-g -O2"
PKG_CONFIG="pkg-config"
GOGCCFLAGS="-fPIC -m64 -pthread -fmessage-length=0 -fdebug-prefix-map=/tmp/go-build870443577=/tmp/go-build -gno-record-gcc-switches"


#有编译相关的，也有依赖相关的，其中 GOPROXY="" 便是要设置成中国的代理的地方
#1.13及其以上版本可以使用-w
go env -w GOPROXY=https://goproxy.cn,direct


# <=1.12版本的要使用系统环境变量来覆盖GOPROXY
# 临时覆盖
export GOPROXY=https://goproxy.cn




# 永久覆盖，也就是写道profile中
sudo touch /etc/profile.d/goproxy.sh

sudo vi /etc/profile.d/goproxy.sh
# 粘贴 export GOPROXY=https://goproxy.cn

source /etc/profile.d/goproxy.sh
```

# 4 离线安装依赖
1. 将依赖包解压放到  /usr/lib/golang/src/ 目录下，比如 依赖的文件夹名字叫 myDependency

2. go install /usr/lib/golang/src/myDependency


