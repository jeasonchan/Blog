@[TOC]
#  1 独立安装Maven
maven一般都有两个setting.xml，内容是描述maven的镜像源：
1. 全局的setting文件，在安装的目录的conf目录中，有个settings.xml文件
2. 用户自己的setting文件，即为 用户/.m2/settings.xml
一般情况下，默认使用全局配置，自己修改的推荐修改全局配置，单项目有特殊要求就用用户配置。
# 2 idea自带的Maven
idea自带maven，大版本是3.x，是以插件的形式安装idea中，但是，其实就是一个几乎完整的maven，我的路径是： /home/jeason/下载/idea-IU-192.5728.98/plugins/maven/lib/maven3，然后，该文件夹下面还有：bin、conf等和独立版maven一样的文件夹结构。经过本人亲测，将该bin文件夹配置进系统环境变量后，便可直接在终端中使用maven命令了。
同理，这个插件版的maven全局配置文件就是 /home/jeason/下载/idea-IU-192.5728.98/plugins/maven/lib/maven3/conf下的xml，而用户配置和独立版maven一样，也是.m2/中的xml文件。
对于idea，本身是直接默认使用全局的设置文件，然后idea里maven的设置选项，默认不勾选override，就是默认不使用用户自己的配置文件，由于本人多台电脑，勾选override会导致多台开发机的idea同步配置，导致多台开发机都会默认使用用户配置文件，因此，本人去修改插件maven的全局配置文件。
打开setting文件，在  mirrors  xxxxxx  /mirrors  中间粘贴阿里的镜像描述，最终结果如下：
```xml
  <mirrors>
  
   <mirror>
   <id>alimaven</id>
   <name>aliyun maven</name>
   <url>http://maven.aliyun.com/nexus/content/groups/public/</url>
   <mirrorOf>central</mirrorOf>
    </mirror> 

  </mirrors>
```
或者，在  profiles  xxxxx  profiles  粘贴阿里的仓库描述，最终结果如下：
```xml
<profile>

<repository>
    <id>nexus-aliyun</id>
    <name>Nexus aliyun</name>
    <layout>default</layout>
    <url>http://maven.aliyun.com/nexus/content/groups/public</url>
    <snapshots>
        <enabled>false</enabled>
    </snapshots>
    <releases>
        <enabled>true</enabled>
    </releases>
</repository>

</profile>
```
**注意！注意标签的结构层次，在  mirrors   xxxxxx  /mirrors  里粘贴的时不带s的镜像标签，同理，另一个粘贴的是不带s的profile文件，都是单数，和dependency和dependencies的结构层次同理**

**重启idea，全局配置文件生效！**
