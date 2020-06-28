1、maven shade 插件，设置mainClass等必要字段
2、编写项目的yml文件，其中的端口号、IP、Path等务必清楚
3、编写
bin文件夹：
init.sh  自己写的打包脚本，主要功能是将，maven打包好的jar包、配置文件、待执行的脚本打包。由于此打包脚本涉及到很多文件名，脚本的文件名处理要小心。
run.sh dockerfile调用，被打包脚本cp到目标文件夹
setenv.sh dockerfile调用，被打包脚本cp到目标文件夹
stop.sh  dockerfile调用，被打包脚本cp到目标文件夹 
4、Dockerfile
From 所依赖的镜像必须是dock库中有的，确定镜像名和版本号
确认脚本名称，防止无法执行脚本
5、大型的PaaS平台可能还会使用带有蓝图的部署方式，编写蓝图时，涉及微服务的名称、端口，务必和上面相关的参数保持一直，比如自己的server的yml中的端口、名称什么的
6、上传dockerfile文件和使用打包脚本打好的包到容器宿主机

建议：
dockerFile中有Docker基础镜像系统的版本号=蓝图中image版本号=上传镜像时的版本号，方便版本管理，要是只用一个基础镜像，自己娱乐娱乐，可以不用统一版本号。
