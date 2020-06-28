@[TOC]
# 1 背景
重新安装ubuntu是选择的LVM磁盘管理方式进行安装，这种方式稍微牺牲了一点IO性能，但是换来快速茶创建磁盘快照和的动态调整个各分区大小，且在系统运行过程中，各个分区用的其实是同一个物理partion，不再像之前纠结各个分区的大小，感觉十分方便。但是，安装后，使用发现，CPU基本一直满负荷，甚至看视频时一直是睿频状态，这就导致笔记本的发热及其夸张，为了限制cpu的频率，尝试了多种方法，包括安装单独的cpu管理软件，但是，最终实践下来，结合cpu状态切换、简洁易用程度等各方面，发现以gnome-shell extension为安装形式的cpu power manger为最佳实践。
# 2 gnome-shell 拓展简介
linux和windows不同，linux的各个发行版可以自己更换shell，shell就是图形界面，gnome是众多shell中的一个，ubuntu18自带的shell就是gnome，gnome shell自带/集成了很多自己的应用，比如 gedit就是gnome中一个不错的文本编辑器，同时，gnome的设置中，可以自己修改的内容十分有限，为了丰富用户的功能定制，gnome-shell extension 出现了，顾名思义，就是gnome桌面的扩展程序，安装拓展后，就可以以图形界面的方式去修改gnome桌面中的参数。
为什么要强调以图形界面的方式去修改gnome桌面的参数？因为，命令行是万能的，只要你知道那个参数的key是什么，你都可以用命令行去修改，拓展是只是提供了图形界面方便你去修改。

如果安装gnome桌面的拓展呢？
sudo apt install extensionName或者ubuntu软件中心的附件组件中或者直接在软件中心搜索。
还有一种方式，在特定的浏览器中安装拓展，然后去gnome网站中直接下载添加。
个人推荐apt install或者软件中心安装，方便。

如何管理gnome桌面的拓展呢？
gnome拓展不像应用程序，在launch中有图标。因此，需要一个应用程序，统一进入、管理gnome桌面拓展——gnome tewak tool，直接在应用商店中搜索tewak即可，下载安装后，图标和名称是“优化”，它的功能不仅仅限于管理拓展，可以说是常用的自定义设置的图形界面合集，把一些比较常用的设置项，以图形界面的方式集合到一个应用程序中了，不用再敲命令行去修改了，十分方便。
其中，扩展一栏就是管理你当前安装的所有的gnome shell extension，可以单独禁用、启用某个扩展，也有一个总开关，启用、禁用所有的扩展。如下图所示：
![gnome tweak tool界面](https://img-blog.csdnimg.cn/20190913230346522.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2plYXNvbl9jaGFuX3pqdQ==,size_16,color_FFFFFF,t_70)
# 3 cpu power manager
应用商店里搜索cpu power manager，下载安装，然后进入tweak编辑该扩展的设置项，该扩展安装后在顶栏有显示，且可以直接在顶栏切换cpu的场景，比如自己的命名的平衡模式，各个场景可以自己定义频率的百分比和是都开关睿频。
使用了这个功能后，关闭睿频后，温度下降明显，再进一步降低频率，限制到10%，甚至都可以24小时开机，在家里当一个server了。
自己的场景定义如图：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190913231159136.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2plYXNvbl9jaGFuX3pqdQ==,size_16,color_FFFFFF,t_70)
