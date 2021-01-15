# 0 背景
比较闲，想尝试控制任务管理中CPU的负载曲线，画个龙画个虎啥的，这就涉及到的多核多线程操作系统中的CPU亲和性了，对亲和性这块不熟，想查看一下文档，那自然是用man查看linux的系统调用文档了，我们都知道可以用man命令来查看linux命令的用法，但是却不知道怎么查看系统调用函数的用法。

# 1 man的使用方法



```bash
man man

# man的使用手册如下
MAN(1)                                               手册分页显示工具                                               MAN(1)

名称
       man - 在线参考手册的接口

概述
       man [-C file] [-d] [-D] [--warnings[=warnings]] [-R encoding] [-L locale] [-m system[,...]] [-M path] [-S list] [-e
       extension] [-i|-I] [--regex|--wildcard] [--names-only] [-a] [-u] [--no-subpages] [-P pager] [-r  prompt]  [-7]  [-E
       encoding]  [--no-hyphenation]  [--no-justification]  [-p  string]  [-t]  [-T[device]]  [-H[browser]] [-X[dpi]] [-Z]
       [[section] page[.section] ...] ...
       man -k [apropos 选项] 正则表达式 ...
       man -K [-w|-W] [-S list] [-i|-I] [--regex] [章节] 词语 ...
       man -f [whatis 选项] 页 ...
       man -l [-C 文件] [-d] [-D] [--warnings[=警告]] [-R 编码] [-L 区域] [-P 分页程序]  [-r  提示]  [-7]  [-E  编码]  [-p
       字符串] [-t] [-T[设备]] [-H[浏览器]] [-X[dpi]] [-Z] 文件 ...
       man -w|-W [-C 文件] [-d] [-D] 页 ...
       man -c [-C 文件] [-d] [-D] 页 ...                                                                                      
       man [-?V]                                                                                                              
                                                                                                                              
描述                                                                                                                          
       man     是系统的手册分页程序。指定给    man    的    页    选项通常是程序、工具或函数名。程序将显示每一个找到的相关    
       手册页。如果指定了 章节，man 将只在手册的指定 章节 搜索。默认将按预定的顺序查找所有可用的 章节 (默认是“1 1p 8  2  3
       3p   4   5   6   7   9   0p   n   l  p  o  1x  2x  3x  4x  5x  6x  7x  8x”，除非被  /etc/man_db.conf  中的  SECTION
       指令覆盖)，并只显示找到的第一个 页，即使多个 章节 中都有这个 页面。

       下表显示了手册的 章节 号及其包含的手册页类型。

       1   可执行程序或 shell 命令
       2   系统调用(内核提供的函数)
       3   库调用(程序库中的函数)
       4   特殊文件(通常位于 /dev)
       5   文件格式和规范，如 /etc/passwd
       6   游戏
       7   杂项(包括宏包和规范，如 man(7)，groff(7))
       8   系统管理命令(通常只针对 root 用户)
       9   内核例程 [非标准

       一个手册 页面 包含若干个小节。

       小节名称通常包括     NAME,     概述(SYNOPSIS),     配置(CONFIGURATION),      描述(DESCRIPTION),      选项(OPTIONS),
       退出状态(EXIT STATUS),   返回值(RETURN VALUE),   错误(ERRORS),   环境(ENVIRONMENT),   文件(FILES),  版本(VERSIONS),
       符合标准(CONFORMING TO), 注(NOTES), 缺陷(BUGS), 示例(EXAMPLE), 作者(AUTHORS), 和 亦见(SEE ALSO).

       以下规范适用于 概述(SYNOPSIS) 小节，也可作为其他小节的指南。

       加粗文本       按原样显示。
       倾斜文本       用相应的参数进行替换。
       [-abc]         “[ ]” 内的任意/全部参数都是可选的。
       -a|-b          以“|”分隔的选项可以一起使用。
       参数 ...       参数 可以重复。
       [表达式] ...   “[ ]”内的整个 表达式 可以重复。

       实际渲染的效果可能因输出设备而异。例如，在终端中 man 程序通常无法渲染出斜体，这时一般会以下划线或彩色文字代替。

       程序和函数说明应该是一个可以匹配所有可能用法的模式(pattern)。有些情况下，建议按此手册页              概述(SYNOPSIS)
       小节所显示的分别陈述几种互斥的用法。

示例
       man ls
           显示 项目 (程序)  ls 对应的手册页

       man man.7
           Display the manual page for macro package man from section 7.

```

如果只想查看系统调用中的相关的函数，就使用 man 2 read 或者 man 3 read，如果直接使用man read那就会显示用户命令read的文档，例子如下：

```bash
man read|cat

BASH_BUILTINS(1)                                                                 General Commands Manual                                                                 BASH_BUILTINS(1)

NAME
       bash,  :,  ., [, alias, bg, bind, break, builtin, caller, cd, command, compgen, complete, compopt, continue, declare, dirs, disown, echo, enable, eval, exec, exit, export, false,
       fc, fg, getopts, hash, help, history, jobs, kill, let, local, logout, mapfile, popd, printf, pushd, pwd, read, readonly, return, set, shift, shopt, source, suspend, test,  times,
       trap, true, type, typeset, ulimit, umask, unalias, unset, wait - bash built-in commands, see bash(1)

BASH BUILTIN COMMANDS
       Unless  otherwise noted, each builtin command documented in this section as accepting options preceded by - accepts -- to signify the end of the options.  The :, true, false, and
       test builtins do not accept options and do not treat -- specially.  The exit, logout, return, break, continue, let, and shift builtins accept and process arguments beginning with
       - without requiring --.  Other builtins that accept arguments but are not specified as accepting options interpret arguments beginning with - as invalid options and require -- to
       prevent this interpretation.


#==============================================
man 2 read|cat


READ(2)                                          Linux Programmer's Manual                                         READ(2)

NAME
       read - read from a file descriptor

SYNOPSIS
       #include <unistd.h>

       ssize_t read(int fd, void *buf, size_t count);

DESCRIPTION
       read() attempts to read up to count bytes from file descriptor fd into the buffer starting at buf.

       On  files that support seeking, the read operation commences at the file offset, and the file offset is incremented
       by the number of bytes read.  If the file offset is at or past the end of file,  no  bytes  are  read,  and  read()
       returns zero.

       If  count  is  zero, read() may detect the errors described below.  In the absence of any errors, or if read() does
       not check for errors, a read() with a count of 0 returns zero and has no other effects.


```
一个是BASH_BUILTINS(1)，一个是
