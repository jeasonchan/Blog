# 1 背景
最近在使用一款小众的Linux发行版，需要配置系统的全局代理，并安装jetbrains家的IDE，记录总结一下。


# 2 配置全局系统代理
```bash
# 先查看一下系统中是否已经包含代理相关的环境变量
env|grep -i proxy

#我的输出是：
#https_proxy=http://proxy.abc.com.cn:80
#http_proxy=http://proxy.abc.com.cn:80
#no_proxy=localhost,127.0.0.0/8,::1,10.0.0.0/8
#all_proxy=socks://proxy.abc.com.cn:80
#ftp_proxy=http://proxy.abc.com.cn:80

#可见该发行版默认配置了全局的系统代理

#修改全局系统代理，将代理地址和端口改成自己期望的
sudo vi /etc/profile

#使配置生效
source /etc/profile

#验证一下当前的系统代理
env|grep -i proxy
```

# 3 安装IDEA并添加菜单图标
本人使用过的Linux的IDEA有三种安装方式：

1. ubuntu里面，用snap install即可，这个发行版好象是基于centos8，不自带snap包管理器，不可行
2. 使用jetbrains提供的Toolbox安装idea，但是，Linux的toolbox提供的是appimage格式，在ubuntu上安装正常，但是此发行版安装是报gpu crash之类的错误……不可行
3. 直接下载idea的tar包安装

此发行版最终采用的第三种方式，直接用下载tar包安装

## 3.1 下载并解压
下载完idea的压缩包后，解压到一个安全的位置（文件夹的全路径不要随意更改），之后idea运行都要直接使用这些解压后的文件。

## 3.2 安装
```bash
cd /path/to/your/idea解压后的文件夹/bin

# 执行安装/启动脚本
./idea.sh

# 然后会出现图形界面，设置完就算安装/初始化 完成了
```

之后每次启动idea都是运行idea.sh这个脚本即可，但是，显然没有点击任务栏的图标方便，下面介绍将在任务栏和开始菜单添加程序启动入口

## 3.3 添加开始菜单启动入口
开始菜单一般叫launcher（启动器），启动器里面程序图标本质上就是.desktop后缀的文件，该发行版的启动器的图标都在 /usr/share/applications 下面，因此，要想在开始菜单中增加idea，只需要在/usr/share/applications  中增加  idea.desktop 文件即可。

1. sudo touch /usr/share/applications/idea.desktop

使用sudo是因为，/usr/share/applications 文件夹是root用户属组的，权限是755，也就是，其他组的用户只有读和执行的权限，即，**只有root用户可以增加启动图标，其他所有用户都可以双击图标启动程序**。

2. 编辑上面新增的图标文件 sudo vi /usr/share/applications/idea.desktop

插入以内容，对图标属性进行描述;
```
[Desktop Entry]
Name=idea
Exec=/path/to/your/idea-IU-202.6397.94/bin/idea.sh
Type=Application
Icon=/path/to/your/idea-IU-202.6397.94/bin/idea.png
Terminal=false
```
3. 刷权限，和其他图标权限一致 sudo chmod 755 /usr/share/applications/idea.desktop

3. 注销并重新登录，就可以看到开始菜单中出现了idea图标（有时候不需要注销就能直接看到新增的图标）

# 4 补充
打开一个文件浏览器，然后使用 ps -ef|grep file ，可以发现该发行版文件浏览器的可执行文件名字叫  ABC-filexxx，再用ps -ef|grep ABC查看一下，发现该发行版的桌面就叫ABC，使用man ABC-filexxx查看一下，能发现ABC所有的系统组件的名字，命名都比较统一。


