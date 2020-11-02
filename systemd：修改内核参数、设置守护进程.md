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

<!-- 从指定的文件加载系统参数，如不指定即从/etc/sysctl.conf中加载 
所以，很多人根本没改/etc/sysctl.conf，却最后也执行-p，毫无卵用-->
# sysctl -p ;

<!-- 重启systemd-sysctl守护进程，确保内核参数都生效 -->
systemctl restart systemd-sysctl.service
```

## 2.2 新建守护进程（开机启动时就禁用透明大页）
想临时禁用透明大页直接执行：

```
echo never > /sys/kernel/mm/transparent_hugepage/enabled
```
但是，系统重启后，参数又会变为默认的always参数，为了实现永久修改，就设置一个开机启动守护进程，系统启动就进行修改。

守护进程也是通过xxxx.service文件描述的，位置位于/etc/systemd/system/中，**开机时，Systemd只执行/etc/systemd/system/目录里面的service文件。但是，systemctl好像能识别到/usr/lib/systemd/system/里的service文件，要想开机执行，只需要将/usr/lib/systemd/system/里的软连接到/etc/systemd/system/中**。

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


systemd的中文文档   http://www.jinbuguo.com/systemd/systemd.service.html


# 3.1 守护进程自动重启

**Restart**

当服务进程 正常退出、异常退出、被杀死、超时的时候， 是否重新启动该服务。 所谓"服务进程" 是指 ExecStartPre=, ExecStartPost=, ExecStop=, ExecStopPost=, ExecReload= 中设置的进程。 **当进程是由于 systemd 的正常操作(例如 systemctl stop|restart)而被停止时， 该服务不会被重新启动。** 所谓"超时"可以是看门狗的"keep-alive ping"超时， 也可以是 systemctl start|reload|stop 操作超时。


**WatchdogSec=**

设置该服务的看门狗(watchdog)的超时时长。 看门狗将在服务成功启动之后被启动。 该服务在运行过程中必须周期性的以 "WATCHDOG=1" ("keep-alive ping")调用 sd_notify(3) 函数。 如果在两次调用之间的时间间隔大于这里设定的值， 那么该服务将被视为失败(failed)状态， 并会被强制使用 WatchdogSignal= 信号(默认为 SIGABRT)关闭。 **通过将 Restart= 设为 on-failure, on-watchdog, on-abnormal, always 之一， 可以实现在失败状态下的自动重启该服务。** 这里设置的值将会通过 WATCHDOG_USEC= 环境变量传递给守护进程， 这样就允许那些支持看门狗的服务自动启用"keep-alive ping"。 如果设置了此选项， 那么 NotifyAccess= 将只能设为非 none 值。 如果 NotifyAccess= 未设置，或者已经被明确设为 none ， 那么将会被自动强制修改为 main 。 如果未指定时间单位，那么将视为以秒为单位。 例如设为"20"等价于设为"20s"。 默认值"0"表示禁用看门狗功能。


**RestartSec=**

置在重启服务(Restart=)前暂停多长时间。 默认值是100毫秒(100ms)。 如果未指定时间单位，那么将视为以秒为单位。 例如设为"20"等价于设为"20s"


可见，watchdog算是一种监控守护进程是否挂掉的功能，restart是直接执行重启的，restartSec是执行重启前的延迟时间，因此，为了实现守护进程的自动重启，可以在[service]中添加以下描述：

```service
WatchdogSec=180s

Restart=always

```

或者

```service
Restart=on-failure

RestartSec=1
```
# 4 补充
1. 以systemd作为引导的Linux系统，内核参数的修改和守护进程定制都是通过service文件进行定制的；sysctl是专门管理内核参数的守护进程，全局限定各种资源的上限和模式；在各个守护进程的service文件里的[service]声明各种值，是针对当前守护进程的，比如，进程的句柄数、线程数什么的，像透明页、开启ipv6这种全局的内核参数，则必须通过sysctl进行修改，并不是针对某个进程的。

2. 中间件部署配置的修改原则：

（1）优先使用默认的配置文件，相关的配置直接在自动生成的配置中修改

（2）配置文件中的路径不要修改，但可以软链接到大容量或者高性能硬盘

（3）需要修改service文件时，不要动原始的service文件，而是应该追加到/etc/systemd/system/xxxxx.service.d/xxxxx.conf中，用追加的旧配置覆盖新配置


