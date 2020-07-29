# 1 背景
新项目开荒，以daemon（守护进程）的方式安装redis，启动后redis日志中有告警：

```
12516:M 22 Jul 2020 17:22:28.039 # WARNING: The TCP backlog setting of 511 cannot be enforced because /proc/sys/net/core/somaxconn is set to the lower value of 128.
12516:M 22 Jul 2020 17:22:28.039 # Server initialized
12516:M 22 Jul 2020 17:22:28.039 # WARNING overcommit_memory is set to 0! Background save may fail under low memory condition. To fix this issue add 'vm.overcommit_memory = 1' to
/etc/sysctl.conf and then reboot or run the command 'sysctl vm.overcommit_memory=1' for this to take effect.
12516:M 22 Jul 2020 17:22:28.039 # WARNING you have Transparent Huge Pages (THP) support enabled in your kernel. This will create latency and memory usage issues with Redis. To fi
x this issue run the command 'echo never > /sys/kernel/mm/transparent_hugepage/enabled' as root, and add it to your /etc/rc.local in order to retain the setting after a reboot. Re
dis must be restarted after THP is disabled.
```

总结下来就是三点告警：

1、内核参数，端口的最大监听数小于redis中配置的监听数

2、内核参数，需要设置vm.overcommit_memory = 1

3、内核参数，修改transparent_hugepage（透明大页）为nerver

前两个的可以直接永久修改的，最后一个透明大页的永久修改需要在系统启动时进行设置，因此，最终使用systemd这一套即可，既可以动态修改并持久化内核参数，也可以设置守护进程实现开机修改某些参数。

# 2 实际操作步骤
守护线程文件位置   /etc/systemd/system/

默认的内核参数配置文件位置   /etc/sysctl.conf

自定义的内核参数配置文件位置    /usr/lib/sysctl.d/

## 2.1 修改内核参数
Linux中一切皆文件，内核参数同样也是从文件中读取。先通过文件查看一下内核参数信息：

```bash
[root@cloud-07 kernel]# cat /proc/sys/net/core/somaxconn
1280
```
可见，当前最大的连接数是1280。

查看一下内核参数的默认配置文件：

```
[root@cloud-07 etc]# cat /etc/sysctl.conf
# System default settings live in /usr/lib/sysctl.d/00-system.conf.
# To override those settings, enter new settings here, or in an /etc/sysctl.d/<name>.conf file
#
# For more information, see sysctl.conf(5) and sysctl.d(5).
fs.file-max=2024000
vm.max_map_count=362144
```

可见，系统建议我们在/usr/lib/sysctl.d/目录下的，用类似与Spring的yml配置文件的方式，用字典序靠后的配置文件覆盖默认的配置文件，因此：

```
cd /usr/lib/sysctl.d/;
touch 99-redis.conf;

<!-- 向文件中写入内核参数配置 -->
echo "vm.overcommit_memory = 1;net.core.somaxconn=1280" > 99-redis.conf;

<!-- 从指定的文件加载系统参数，如不指定即从/etc/sysctl.conf中加载 -->
sysctl -p ;

<!-- 重启systemd-sysctl守护进程，确保内核参数都生效 -->
systemctl restart systemd-sysctl.service
```

## 2.2 新建守护进程（开机启动时就禁用透明大页）
想临时禁用透明大页直接执行：

```
echo never > /sys/kernel/mm/transparent_hugepage/enabled
```
但是，系统重启后，参数又会变为默认的always参数，为了实现永久修改，就设置一个开机启动守护进程，系统启动就进行修改。

守护进程也是通过xxxx.service文件描述的，位置位于/etc/systemd/system/中，

```
cd /etc/systemd/system/;
touch disable-THP.service;

<!-- 
输入以下service内容，实现开机执行禁用脚本

[Unit]
Description=Disable Transparent Huge Pages (THP)
[Service]
Type=simple
ExecStart=/bin/sh -c "echo 'never' > /sys/kernel/mm/transparent_hugepage/enabled && echo 'never' > /sys/kernel/mm/transparent_hugepage/defrag"
[Install]
WantedBy=multi-user.target

 -->
vi disable-THP.service;

<!-- 重新加载一遍所有的xxxx.service文件 -->
systemctl daemon-reload

<!-- 启动守护进程 -->
systemctl start disable-thp

<!-- 允许守护进程开机自启动 -->
systemctl enable disable-thp
```

最后，systemctl restart redis 重启redis进程，查看redis日志确认告警是消除。

```
systemctl restart redis;
tail -100 /var/log/redis/redis.log
```

# 3 systemd详解
上文的systemd相关的知识较少，详见参考文档补充相关知识。

参考文档：

Systemd 入门教程：命令篇      http://www.ruanyifeng.com/blog/2016/03/systemd-tutorial-commands.html


