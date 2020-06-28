方案概述：
1. 将linux子系统（简称wsl）安装到非系统盘，参考过 <https://blog.csdn.net/fleaxin/article/details/88587081>、<https://www.jianshu.com/p/a9863b3eaed4>
2. wsl中安装docker，参考过<https://www.jianshu.com/p/97d16b68045f>、<https://blog.csdn.net/world_snow/article/details/79625341>
# 安装wsl到非系统盘
最终目的很明确，在一个非系统盘的文件夹中运行安装ubuntu可执行文件，即，运行 ubuntuXXXX.appx中的ubuntuXXXX.exe。步骤如下：
```powershell
# 管理员身份运行powershell

#用命令行启用WSL；也可以使用控制面板中的添加windows功能，给 windows中的linux子系统 选型打钩
Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Windows-Subsystem-Linux

#给wsl创建安装目录，并切换到安装目录下面
New-Item D:\WSL -ItemType Directory
Set-Location D:\WSL

#使用命令行下载安装程序
#也可以用迅雷下载 Ubuntu 18.04 https://aka.ms/wsl-ubuntu-1804 ，下载完重名为Ubuntu.appx，并拷贝到 D:\WSL
Invoke-WebRequest -Uri https://aka.ms/wsl-ubuntu-1804 -OutFile Ubuntu.appx -UseBasicParsing

#文件名改后缀，方便解压；也可以直接手动改文件名并手动解压；
Rename-Item .\Ubuntu.appx Ubuntu1804.zip
Expand-Archive .\Ubuntu1804.zip -Verbose

#运行程序；也可以直接双击运行。
cd Ubuntu1804
.\ubuntu1804.exe

#完事儿
```
简单使用说明：
1. powershell输入bash，启动进入ubuntu环境，默认的路径是**windows的当前用户的路径，并不是ubutnu的路径**，"cd  /"可切换到ubuntu根路径，其余操作一样普通的ubuntu一样
2. 和win10使用同一IP，由于保护机制，系统初始化时设置的初始密码无法切换到root账户，使用"sudo passwd"根据提示重设密码，跟原来的密码一样也完全没问题

# wsl中安装docker
在win10中实现docker的方法有多种，前人提出的有：
* Win10 + Docker <-- 本机共享文件夹Volume --> Ubuntu命令行(in WSL)
* Ubuntu --> remote Docker daemon --> Docker in Windows
* Win10浏览器 --> Docker in Windows --> Ubuntu IP --> Ubuntu内服务
但是，为了体验“纯正”的服务器体验，本文采用wsl+wsl中的docker，即，docker及其仓库完全在wsl中，因此，直接使用apt包管理工具安装docker，安装完docker，考虑更换docker拉取的镜像源，大家都说用网易的镜像源……
```bash
sudo apt-get install docker.io
```
安装完运行“docker”命令，第一行就告诉你了配置文件的地址，比如我的是"/home/wsl/.docker"，然后用vi编辑添加下面的，即可将更换为网易的docker镜像源……
**实践最后，发现wsl不支持docker的守护进程，想使用还是要依靠docker for windows！还是只能在wsl中使用client，无法在wsl中启动service……想体验原生的docker还是hyper-v或者虚拟机！= _ =**


