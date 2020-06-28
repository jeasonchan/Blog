@[TOC]
换用Ubuntu18，发现比之前的用的老版本多出老了snap安装方法，又很久不用Ubuntu了，带上新的包管理工具，回顾一下ubuntu安装、卸载、更新、清除软件的方法。
目前来看，共有以下四种方法，从方便安装、更新、卸载的角度来看，推荐的顺序为：应用商店>snap>apt>dpkg>make。
# 基本概念
软件仓库是一组文件，其中包含各种软件及其版本的信息，以及校验和等其他一些详细信息。每个版本的 Ubuntu 都有自己的四个官方软件仓库：
* Main - Canonical 支持的自由开源软件。
* Universe - 社区维护的自由开源软件。
* Restricted - 设备的专有驱动程序。
* Multiverse - 受版权或法律问题限制的软件。
包含软件信息的网址存储在 /etc/apt 目录中的 sources.list 文件中，运行 sudo apt update 命令时，系统将使用APT工具来检查软件仓库并将软件及其版本信息存储在缓存中。当你使用 sudo apt install package_name 命令时，它通过该信息从实际存储软件的网址获取该软件包。
# 应用商店
GUI的操作方式，安装、卸载等最方便，但是，个人感觉，搜索软件时十分缓慢。
软件源有至少有两处：
1. 官方的源，就是百度一下，每个博客都在说的ubuntu发行版的软件源，可以换成百度、网易的源。更换的方式有两种：
* 通过终端修改，以root权限修改/etx/apt/sources.list文件中的源地址，虽然各个都说国外的源很慢，但是，都什么年代了……国外的官方源一点也不算慢啊……目前来看不是很有必要修改了，哪些抄来抄去的博文也是醉了……
* 通过GUI自动修改。打开软件“软件和更新”，设置，可以自动测试并选择的最佳的服务器地址。改完之后，可以在终端里“sudo update”一下，可以发现请求地址已经不是原来的国外源了，我的变成了上海交大的源地址。
2. snap源，18.0.4默认安装了snap-core，可以支持canonical推出的新的打包方式，所有的依赖都时打包在一个包中，安装和卸载时不会有“索引依赖……”之类的控制台打印了，直接在应用商店里搜索应用，可以看到有个标签描述，很多应用的来源都已经是snap了。
3. PPA源，系统--->系统管理--->软件源--->其他软件--->添加--->(输入) ppa:user/ppa-name--->添加源--->关闭。并且，等效于在终端下运行“add-ppa-repository ppa:user/ppa-name”命令。
# snap
参考<https://www.linuxidc.com/Linux/2018-05/152385.htm>
Snap的安装包扩展名是.snap，类似于一个容器，它包含一个应用程序需要用到的所有文件和库（snap包包含一个私有的root文件系统，里面包含了依赖的软件包）。它们会被安装到单独的目录；各个应用程序之间相互隔离。使用snap有很多好处，首先它解决了软件包的依赖问题；其次，也使应用程序更容易管理。snap软件包一般安装在/snap目录下（尚未验证）。
一些常用的snap命令：
```shell
sudo snap list #列出已经安装的snap包
sudo snap find <text to search> #搜索要安装的snap包
sudo snap install <snap name> #安装一个snap包
#更新一个snap包，如果你后面不加包的名字的话那就是更新所有的snap包
#不确定通过install命令是否会安装更新的版本
sudo snap refresh <snap name> 
sudo snap remove <snap name> #删除一个snap包
snap change #查看一下正在进行的change
snap abort 数字 #终止正在进行的snap change，用于终止各种snap异常情况，十分有用！！！
```
当然，snap软件的安装方式远不止这一种，设置可以下载snap直接双击安装，比如opera浏览器，双击opera官网下载的安装包，会自动打开应用商店进行安装。
snap打包的软件甚至能像Windows一样，自身提更新功能，和传统的通过源更新的软件体验相比，体验提升巨大！
# apt和PPA
apt是advanced package tool的简称，目前使用apt根本不需要像以前一样输入完成的“apt-get”之类的，apt官方将全部的常用的命令整合进“apt”当中，直接使用“apt 选项 参数”的结构即可，根本不需要的“apt-get”的方式，时代在变，接受一下apt这个工具的自我变革。apt的常用命令和snap几乎完全相同，不赘述了。
使用PPA安装，即从launchpad<>源安装软件，launchpad相当于是对apt官方源的一个补充，相当于第三方源，个人可以把软件包的传导launchpad中，然后要使用该软件的人，需要先“订阅”该软件的launchpad源，然后update和upgrade该软件。典型的安装过程如下：
```shell
sudo add-apt-repository ppa:webupd8team/java #PPA 的一般形式是： ppa:user/ppa-name
sudo apt update
sudo install 包名
```
PPA推荐Linux中国的文章<https://linux.cn/article-10456-1.html>，强烈推荐，其中讲到了其他基本知识概念。

# dpkg
目前甚至还需要通过这个命令安装的软件，**体验一般都不咋地**，一般都是很久不更新的旧软件了……
但是！！！目前又有很多大厂软件采用deb包的方式进行分发，比如Chrome，Chrome会自动忘apt源列表中新增自己的PPA源，使自身能在apt update中检测到更新。同理，还有微软爸爸的vscode。软件安装后，软件安装过程中会自动添加源。可以在Ubuntu的更新设置——其他软件 中看到相应的第三方源。
# make
通过源码安装，个人反而觉得很有趣，自己的机器编译源码然后使用，虽然体验一般，但是很有趣。
# 总结
linux的软件安装形式，个人感觉更像windows下面的“绿色版”软件，安装一个软件只是将软件拷贝到一个专门放软件包的文件夹当中，然后通过软连接等方式将软件的启动“脚本”展现给用户或者添加到环境变量中（方便终端识别名称），因此，对软件的更新和卸载就是对该软件包的刷新和删除，而配置文件一般只有额外通过purge参数或者命令才能清除。
