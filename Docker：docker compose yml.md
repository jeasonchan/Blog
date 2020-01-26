# 1 背景
使用vscode的docker插件自动生成Dockerfile文件时,除了自动生成了Dockerfile文件外，还自动生成了docker compose yml文件，系统学习一下这个文件的作用。

参考：

[Dockerfile与docker-compose.yml文件的作用及自动部署的实现](https://www.jianshu.com/p/a73ca4bb7dd9)

# 2 为什么使用Docker Compose File？
按照docker官方的建议，每一个容器只启动一个进程，这样便于管理和解耦。而在生产部署的时候，我们的一个应用不太可能只有一个进程，除了代码应用的主进程外，你可能还需要开启reids、mysql、nginx等。也就是说不会只靠一个镜像便能部署完成，所以我们每次部署应用需要同时用多个镜像启动多个容器，操作端口映射、数据卷，完成容器间的通信。如果涉及到分布式和多台服务器，那岂不是每个服务器都得这样操作一次？因此，docker提供了Docker Compose File，可以使用docker-compose.yaml文件，按照特定的语法语句编写指令，**一个文件即可管理多个镜像的部署和端口等操作，实现真证的快速部署。**在不同服务器上部署时，只需要一个docker-compose.yaml文件，便能完成应用的部署操作。

# 3 制作docker-compose.yaml文件
在flask-project目录内创建并编辑docker-compose.yaml文件：vi docker-compose.yaml，编辑内容如下：
```
version: "3.6"
services:
  flask-web:
    build: .
    ports:
        - "5000:9999"
    container_name: flask-web
  redis:
    image: redis
    container_name: redis
```
version：版本注释，不可缺少的字段。

services：该层级下指明使用镜像开启容器的具体配置，是最主要的配置项。

flask-web、redis：自定义的该service名字。

build：Dockerfile（更准确的说应该是上下文路径，并且会使用默认的Dokcerfile作为镜像构建依据）的路径，使用它来创建一个定制的镜像，或者可使用image指定已有镜像。

image：指定使用已有镜像。

ports：开启容器后暴露的端口映射。

container_name:指定开启容器后的容器名。

注意：docker-compose.yaml必须按格式规范来写，最好使用两个空格来表示层级关系。每个参数前面都有一个空格。编写完后使用docker-compose config检查是否有语法错误。

最终的目录层级如下：
```
└── flask-project
    ├── docker-compose.yaml
    ├── Dockerfile
    └── flask-web-code
        ├── app.py
        └── requirements.txt
```

# 4 最终部署使用
使用docker-compose config检查语法是否错误

使用docker-compose up -d后台启动服务，过程中会自动根据Dockerfile创建（多个services的）镜像，并且按要求启动服务

使用docker-compose logs检查运行状态

检查项目是否正常运行：curl 127.0.0.1:5000，每次访问能收到变化的数据证明项目部署已经成功完成。
