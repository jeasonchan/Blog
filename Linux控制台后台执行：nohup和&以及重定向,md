# 1 背景
最经买了阿里云主机，没有图形界面，想运行程序全靠ssh到server，然后使用命令行启动。使用命令行启动还可以分为两种：
* “异步执行”，比如，使用docker run命令，container会自动在后台运行，ctrl+c或者ssh/终端，都不会中断程序运行。可以使用docker ps或者docker stats查看容器的运行状态
* “同步执行”，会占用当前控制台，ctrl+c或者退出ssh/当前控制台都会中断程序执行。
使用docker部署、管理起来固然方便，但是，有时候还是有直接运行的需求，急需一种方法让运行的程序不“霸占”当前的控制台，转到后台运行。

因此，可以使用：
```bash
nohup command command的参数 &
```
# 2 nohup
用途：不挂断地运行命令。

语法：nohup Command [ Arg ... ] [　&]。最后的[ &]一般是一起使用的，是单独的一个“关键字”

描述：nohup 命令运行由 Command 参数和任何相关的 Arg 参数指定的命令，忽略所有挂断（SIGHUP）信号。结尾的 & 符号代表这个程序不会受到ctrl+c的影响，依然会在后台运行程序，而控制台可以做其他的事情。

nohup的输出：nohup 命令的输出默认重定向**追加**到当前目录的 nohup.out 文件中。如果当前目录的 nohup.out 文件不可写，输出重定向到 $HOME/nohup.out 文件中。如果没有文件能创建或打开以用于追加，那么 Command 参数指定的命令不可调用。

退出状态：
* 126，可以找到  Command [ Arg ... ]  中的的 Command 命令，但是不能正常执行下去。
* 127，根本找不到  Command [ Arg ... ]  中的的 Command 命令。
* 其他情况，转发 Command [ Arg ... ] 的退出状态
# 2.1 查看运行情况
假如有如下运行场景：
```bash
nohup /usr/local/node/bin/node /www/im/chat.js &
//输出是：
[1] 1234
```
后台运行的进程号是 1234。
有两种方式查看运行的chat.js，jobs和ps -ef。

ps -ef必定是万能的，都能查看。但是，jobs命令只看当前终端生效的，关闭终端后，在另一个终端jobs已经无法看到后台跑得程序了，此时就利用ps了。

# 2.2 输出重定向
先看例子：
```bash
$ nohup echo 23333 >c://test.out 2>&1 &
[1] 873
[1]+  Done                    nohup echo 23333 > c://test.out 2>&1
```
做的事情，实际上就是将233333字符串和标准错误都输出到c://test.out中

在shell中，文件描述符通常是:STDIN标准输入，STDOUT标准输出和STDERR标准错误输出，分别用0、1、2 表示，上面的例子中，标准输出重定向到c://test.out中，（可能出现的）标准错重定向到&1中（同名文件，也就是c://test.out）。
