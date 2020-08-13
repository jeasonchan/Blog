# 1 背景

参考文档：

自身特点及使用场景：   

https://blog.csdn.net/tanqian351/article/details/81744970   

https://blog.csdn.net/justlpf/article/details/80392944


通过yum安装和配置    https://www.jianshu.com/p/d3b31b7aa182

更新MongoDB用户密码、用户权限    https://jingyan.baidu.com/article/d169e18609d989436611d82e.html

MongoDB Change Stream：简介、尝试与应用（体验详细）   https://www.cnblogs.com/xybaby/p/9464328.html

MongoDB Change Stream初体验（体验简略）    https://mongoing.com/change-stream

MongoDB基本交互语法   https://blog.csdn.net/sun5769675/article/details/43734549



## 1.1 简介
基本情况：
* MongoDB 是一个基于分布式 文件存储的NoSQL数据库
* 由C++语言编写，运行稳定，性能高
* 旨在为 WEB 应用提供可扩展的高性能数据存储解决方案

MongoDB特点：
* 模式自由 :可以把不同结构的文档存储在同一个数据库里
* 面向集合的存储：适合存储 JSON风格文件的形式
* 完整的索引支持：对任何属性可索引
* 复制和高可用性：支持服务器之间的数据复制，支持主-从模式及服务器之间的相互复制。复制的主要目的是提供冗余及自动故障转移
* 自动分片：支持云级别的伸缩性：自动分片功能支持水平的数据库集群，可动态添加额外的机器
* 丰富的查询：支持丰富的查询表达方式，查询指令使用JSON形式的标记，可轻易查询文档中的内嵌的对象及数组
* 快速就地更新：查询优化器会分析查询表达式，并生成一个高效的查询计划
* 高效的传统存储方式：支持二进制数据及大型对象（如照片或图片）

# 2 安装
支持自己下载tar包安装，解压后，自己创建配置文件、创建守护进程文件，然后启动即可。对于centos，官方推荐的安装方式是直接yum install，配置文件、守护进程文件一条龙完事儿。

MongoDB官方源中，mongodb-org是MongoDB元数据包，安装时自动安装下面四个组件包：
1. mongodb-org-server: 包含MongoDB守护进程和相关的配置和初始化脚本。
2. mongodb-org-mongos: 包含mongos的守护进程。
3. mongodb-org-shell: 包含mongo shell。
4. mongodb-org-tools: 包含MongoDB的工具： mongoimport, bsondump, mongodump, mongoexport, mongofiles, mongooplog, mongoperf, mongorestore, mongostat, and mongotop。

由于服务器一般会直连外网，无法直接从yum的repo里下载，可根据需要手动下载一些RPM包后sftp上去，再进行安装：

```
wget https://repo.mongodb.org/yum/redhat/7/mongodb-org/4.0/x86_64/RPMS/mongodb-org-server-4.0.5-1.el7.x86_64.rpm
wget https://repo.mongodb.org/yum/redhat/7/mongodb-org/4.0/x86_64/RPMS/mongodb-org-shell-4.0.5-1.el7.x86_64.rpm
wget https://repo.mongodb.org/yum/redhat/7/mongodb-org/4.0/x86_64/RPMS/mongodb-org-tools-4.0.5-1.el7.x86_64.rpm
wget https://repo.mongodb.org/yum/redhat/7/mongodb-org/4.0/x86_64/RPMS/mongodb-org-mongos-4.0.5-1.el7.x86_64.rpm
```

以root用户安装后，用systemctl start mongodb尝试启动，启动报错，查看日志：

```
# 日志一般都在这个目录下
tail -200 /var/log/mongodb/mongod.log

# 报错信息如下
2020-07-30T10:55:34.965+0800 I CONTROL  [main] ***** SERVER RESTARTED *****
2020-07-30T10:55:34.973+0800 I CONTROL  [main] ERROR: Cannot write pid file to /var/run/mongodb/mongod.pid: Permission denied
```

加权限即可，我是无脑加的 777 权限……最终成功启动。但是，官方文档有一句话：By default, MongoDB runs using the mongod user account。可见，需要为上面的pid文件更改拥有者：

```
chown -R mongod:mongod /var/run/mongodb
```

## 2.1 稍微修改配置
安装好了以后，会生成一个 /etc/mongod.conf 的配置文件，配置了 mongoDB 默认的配置。

和ES一样，默认配置的网络服务绑定地址为 127.0.0.1 ，也就意味着默认情况下只能本机连接 mongoDB。

**如果配置自己单独配置了日志、数据路径，要用chown -R mongod:mongod \<dir\> 给相关的目录加权限。**



# 3 单机部署复制集
## 3.1 创建另一个运行实例
查看已经运行的MongoDB：

```
# 两种方式查看，ps更加通用；systemctl查看守护进程形式运行的MongoDB
[root@searchcode-02 mongodb_salve]# ps -ef |grep mongo
root      7308 28084  0 11:57 pts/8    00:00:00 grep --color=auto mongo
mongo    14687     1  0 10:58 ?        00:00:13 /usr/local/mongodb/bin/mongod --auth --config /etc/mongodb/mongod.conf

[root@searchcode-02 mongodb_salve]# systemctl status mongodb
● mongodb.service - mongodb
   Loaded: loaded (/etc/systemd/system/mongodb.service; enabled; vendor preset: disabled)
   Active: active (running) since Thu 2020-07-30 10:58:11 CST; 59min ago
  Process: 14683 ExecStart=/usr/local/mongodb/bin/mongod --auth --config /etc/mongodb/mongod.conf (code=exited, status=0/SUCCESS)
 Main PID: 14687 (mongod)
   CGroup: /system.slice/mongodb.service
           └─14687 /usr/local/mongodb/bin/mongod --auth --config /etc/mongodb/mongod....
```

可见，MongoDB启动本质上就是：

```
/usr/local/mongodb/bin/mongod --auth --config /etc/mongodb/mongod.conf
```
要在同一个机器再部署另一个实例，只需要更换配置文件即可。

为了不影响已经以守护进程运行的实例，新建一个文件夹，将配置文件、pid File、数据库路径都放在其中。


```
mkdir mongodb_salve
cp /etc/mongodb/mongod.conf  ./mongodb_salve

# 修改的一些路径，指向mongodb_salve目录下的一些路径、文件
vi mongod.conf


# 最终配置文件和mongodb_salve目录如下
[root@searchcode-02 mongodb_salve]# cat mongod.conf
systemLog:
   destination: file
   path: "/root/mongodb_salve/systemLog/mongod.log"
   logAppend: true
   logRotate: rename
   timeStampFormat: iso8601-local
storage:
   dbPath: "/root/mongodb_salve/dbpath"
   journal:
      enabled: true
processManagement:
   fork: true
   pidFilePath: "/root/mongodb_salve/pidFilePath/mongod.pid"
net:
   bindIp: 0.0.0.0
   port: 27018
setParameter:
   enableLocalhostAuthBypass: true
#security:
#   keyFile: "/etc/mongodb/.keyFile"
#   authorization: enabled
#replication:
#   oplogSizeMB: 520
#   replSetName: test-repl
#   secondaryIndexPrefetch: all
#   enableMajorityReadConcern: true

[root@searchcode-02 mongodb_salve]# ll
total 12
drwxr-xr-x 4 mongod mongod 4096 Jul 30 11:52 dbpath
-rw-r--r-- 1 mongod mongod  654 Jul 30 11:50 mongod.conf
drwxr-xr-x 2 mongod mongod   31 Jul 30 11:49 pidFilePath
-rw-r--r-- 1 mongod mongod   87 Jul 30 11:47 start-slave.sh
drwxr-xr-x 2 mongod mongod   31 Jul 30 11:40 systemLog

# 务必赋予mongod用户对于mongodb_salve的读写权限
chown -R mongod:mongod ./mongodb_salve
```




## 3.2 修改配置文件构成复制集关系
配置复制集的分两个步骤：

1、修改mongo实例的配置文件，正确配置keyFile，并将所有的mongo实例启动起来

2、进入主节点，通过命令行初始化整个复制集

### 3.2.1 修改配置
修改配置文件，明确配置文件里的keyFile，如下：
```
systemLog:
   destination: file
   path: "/var/log/mongodb/mongod.log"
   logAppend: true
   logRotate: rename
   timeStampFormat: iso8601-local
storage:
   dbPath: "/var/lib/mongodb"
   journal:
      enabled: true
processManagement:
   fork: true
   pidFilePath: "/var/run/mongodb/mongod.pid"
net:
   bindIp: 0.0.0.0
   port: 27017
setParameter:
   enableLocalhostAuthBypass: true
security:
   keyFile: "/etc/mongodb/.keyFile"
   authorization: enabled
replication:
   oplogSizeMB: 520
   replSetName: test-repl
   secondaryIndexPrefetch: all
   enableMajorityReadConcern: true
```

所有的要组成复制集的mongo实例要**共用同一个keyFile**，且keyFile有如下要求：

1. 权限不能太开放，要不然mongo拒绝启动，我是chmod 100 /etc/mongodb/.keyFile
2. 字母、数字、空格的组合，不能用下划线

### 3.2.2 初始化复制集 集群

```
# 使用mongo的命令行工具连接打算设置主节点的mongo实例
mongo

# 新建一个变量，声明 复制集ID、各个成员的连接串、复制集的等级
rsconfig = {_id: 'wic-test-repl', members: [ {_id: 0, host: '127.0.0.1:27017',priority:1}, {_id: 1, host: '127.0.0.1:27018'}] }

# 初始化复制集
rs.initiate(config)

# 检测一下
rs.status()
rs.isMaster()
```

在rs.initiate(config)执行完，已经初始化了复制集之后，还可以动态的增加、删除节点；

```
# 进入主节点

<复制集名称>:PRIMARY>  rs.add("192.168.125.119:27020")

<复制集名称>:PRIMARY>  rs.remove("192.168.125.119:27020")
```

注意！！初初始化副本集用的rsconfig对象，连接串千万不要使用localhost，就算是本地的，也尽量使用IP（比如，127.0.0.1），因为，只要有一个是localhost的形式，其余节点也必须是localhost的形式，并且副本集群模式访问时，有一定的问题，localhost会额外识别为一个节点……

### 3.2.3 设置集群账号密码
通过status、isMaster函数确认为复制集部署方式后，增删改查都需要账号密码，**只需要在当前的主节点上添加账号密码**。要先设置管理员帐号，再用以管理员帐号的身份添加用户账号。

#### 3.2.3.1 添加管理员账号

```
# 接入主节点
mongo

# 切换到自带的admin数据库
use admin  

# 创建管理员用户
db.createUser({user:"admin", pwd:"admin", roles:[{role: "userAdminAnyDatabase", db:"admin" }]})

# 最好断开客户端，重启当前的主节点mongo实例
# 然后以带鉴权的方式启动mongo，并登录

```

以**带鉴权的方式**启动MongoDB：mongod --auth --config /path/to/configFile.conf，然后用账号密码登录验证一下。和mysql一样，登录时有两种鉴权方式：

1. 命令行种直接带用户名、密码、数据库名： mongo --port 27017 -u "admin" -p "admin" --authenticationDatabase "admin"

2. 先连接上，再切换库、鉴权：
```
mongo

# 切换数据库
use admin

db.auth("admin", "admin")
# 返回1
```

经过实验，整个复制集启动起来的情况下，在主节点上操作添加用户什么的，用户信息同样都会同步到从节点。

#### 3.2.3.2 添加普通用户账号
过程类似创建管理员账户，只是 role 有所不同。

```
use foo

db.createUser(
  {
    user: "simpleUser",
    pwd: "simplePass",
    roles: [ { role: "readWrite", db: "foo" },
             { role: "read", db: "bar" } ]
  }
)
```
现在我们有了一个普通用户
用户名：simpleUser
密码：simplePass
权限：读写数据库 foo， 只读数据库 bar。

**注意：**
use foo表示用户在 foo 库中创建，就一定要 foo 库验证身份，即用户的信息是跟随数据库foo的(**这点和mysql不一样，mysql有专门的用户信息记录表**)。比如上述 simpleUser 虽然有 bar 库的读取权限，但是一定要**先切换到 foo 库进行身份验证**，然后才能对bar库进行读操作。一上来就use bar，然后db.auth("simpleUser", "simplePass")必然会验证失败。

```
# 正确的身份验证流程，要先切换到foo库
use foo
db.auth("simpleUser", "simplePass")

use bar
show collections
```
#### 3.2.3.3 MongoDB中用户权限的枚举值
* Read：允许用户读取指定数据库
* readWrite：允许用户读写指定数据库
* dbAdmin：允许用户在指定数据库中执行管理函数，如索引创建、删除，查看统计或访问system.profile
* userAdmin：允许用户向system.users集合写入，可以找指定数据库里创建、删除和管理用户
* clusterAdmin：只在admin数据库中可用，赋予用户所有分片和复制集相关函数的管理权限。
* readAnyDatabase：只在admin数据库中可用，赋予用户所有数据库的读权限
* readWriteAnyDatabase：只在admin数据库中可用，赋予用户所有数据库的读写权限
* userAdminAnyDatabase：只在admin数据库中可用，赋予用户所有数据库的userAdmin权限
* dbAdminAnyDatabase：只在admin数据库中可用，赋予用户所有数据库的dbAdmin权限。
* root：只在admin数据库中可用。超级账号，超级权限


从描述看，数据库案例和角色管理是分开的：
* userAdminAnyDatabase 能管理所有库的用户，因为MongoDB，用户信息是在库里的（先切换到库再进行鉴权验证）
* dbAdminAnyDatabase 能管理所有库的数据

更新用户密码，修改与用户权限可查看链接：https://jingyan.baidu.com/article/d169e18609d989436611d82e.html

举例：
```
use admin

<复制集名称>:PRIMARY> db.grantRolesToUser("admin",[role:{"dbAdminAnyDatabase", db:"admin"}])

<复制集名称>:PRIMARY> show users
{
        "_id" : "admin.admin",
        "user" : "admin",
        "db" : "admin",
        "roles" : [
                {
                        "role" : "dbAdminAnyDatabase",
                        "db" : "admin"
                },
                {
                        "role" : "userAdminAnyDatabase",
                        "db" : "admin"
                }
        ],
        "mechanisms" : [
                "SCRAM-SHA-1",
                "SCRAM-SHA-256"
        ]
}

```

可见，admin最终曾为了最厉害的管理员之一，dbAdminAnyDatabase和userAdminAnyDatabase，既能管理所有库中的数据，也能管理所有库中的用户。


#### 3.2.3.4 极其重要的补充
MongoDB默认是以没有鉴权的（没有账号、密码）方式启动的，连上没有鉴权的MongoDB可以进行任意操作。

要想以带鉴权的方式启动MongoDB，有两种方式：

1. 启动命令行中添加 --auth 参数：
```
/use/bin/mongod --auth --config /etc/mongodb/mongodb.conf
```

2. 在配置文件中增鉴权参数，配置文件局部信息如下：

```
security:
   keyFile: "/etc/mongodb/.keyFile"
   authorization: enabled
```

所以了，想对副本集添加鉴权，根据前提条件有以下两种方式：

1. 所有节点无鉴权启动，已经完成了副本集初始化 的情况下：
   1. 在主节点添加用户名密码
   2. 修改配置文件或者修改启动命令，以带鉴权的方式重启各个副本集节点

2. 所有节点无鉴权启动，还未完成副本集初始化 的情况下：
   1. 选定一个节点，确保这个节点的配置文件中无副本集相关的配置，添加用户名密码
   2. 恢复配置文件中的副本集配置，无鉴权重启一下
   3. 初始化副本集，节点可以包含所有节点，也可以仅包含有限节点在一个一个add
   4. 所有节点带鉴权重启


**所以，还是先初始化成副本集，再添加账户密码方便。**


**重点，未成为任何副本集的节点的情况下，如果配置文件中有副本集相关的配置项，无法添加用户和密码，注掉之后就可以了。**

## 3.3 简单的数据操作

```
use admin
db.auth("admin","admin")

# 使用use命令切换到myTest数据库，若没有系统会自动创建
# 如果我们什么也不干，数据库会被系统回收掉
>use myTest

# 创建一个集合，并创建数据
>db.myuser.insert({"name":"yutao", age:24})

# 上面这条命令我们并没有显式创建集合`myuser`
# mongodb会自动帮我们创建。
# 这样数据库和一个集合（类似于表的概念）我们就创建好了
```


程序中连接这个复制集集群时，要遵循标准的连接串：mongodb://[username:password@]host1[:port1][,host2[:port2],...[,hostN[:portN]]][/[database][?options]]

比如：
mongodb://admin:admin@127.0.0.1:27000,127.0.0.1:27001

# 4 分片副本集部署
简要步骤
1. 先搭建n个副本集，当做n个分片使用
2. 配置一下配置文件，router、config、n个副本集都正常运行起来
3. 在router使用mongo连接上，执行 sh.addShard("副本集名称/副本集的MongoDB连接串") 
4. 必须先对某个db开启使用分片，sh.enableSharding("db_name")
5. 再对允许分片的db中的某个集合开启分片，sh.shardCollection("db_name.db_collection", {column_name: 1})，其中column_name必须是某索引的第一列，空集合会自动先根据分片列创建索引，非空集合则需要我们手动为column_name先创建一个索引


关于设置账号密码，两种思路：
1. 一开始都不要设置设置账号密码+无鉴权启动，一切都完毕之后，在mongo router中设置账号密码，再所有的机器带鉴权全部重启一遍
2. 一开始手贱非要先设置账号密码，那就各个分片的副本集、router、config的账号密码保持一致即可
