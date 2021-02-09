# 0 背景
竟然要开始写python了，还是只需要使用api的程度，没啥成就感……而且，一开始就极其厌恶动态类型语言，项目要写个小工具，用来快速修改系统配置，配置的格式是json格式，shell脚本需要用sed或者依赖第三方库，不方便。

选型了一下，还是python方便，自带包括json模块在内的很多方便的模块，开始重新学习python。

由于工具是像命令行一样，需要在终端中直接输入参数，在此学习记录一下解释器引导时对程序的输入参数。

参考文档：
Python解析命令行读取参数 -- argparse模块   https://www.cnblogs.com/arkenstone/p/6250782.html


# 1 代码实践
文件名，tool.py

```py
#!/usr/bin/python

print("hello world!")
```

然后 chomd +x tool.py 再  ./tool.py 就可以直接运行，并在当前命令行中输出hello world。

第一步已实现直接通过命令行运行工具，接下来实现直接读取命令行输入的参数

## 1.1 直接使用sys.argv
```py
import sys
print "Input argument is %s" %(sys.argv)
```

```bash
$ ./main -add 1,2,3 , 2
input argv is ['./main', '-add', '1,2,3', ',', '2']
```
可见，argv和其他语言一样，都是数组，数组第一个元素是路径，经过试验，是python启动时的路径，之后的参数就是以空格作为分隔符，依次放入数组中。

参数较少时，直接自己读取数组元素处理即可，但是，硬编码会比较严重，且增加参数时会比较麻烦，转而使用自带的另一个模块argparse

## 1.2 使用argparse

```py
#!/usr/bin/python

import sys
import argparse

print("input argv is {}".format(sys.argv))

# description参数可以用于插入描述脚本用途的信息，可以为空
parser = argparse.ArgumentParser(description="your script description")

# 添加--verbose标签，标签别名可以为-v，这里action的意思是当读取的参数中出现--verbose/-v的时候
# 参数字典的verbose建对应的值为True，而help参数用于描述--verbose参数的用途或意义。
# 将变量以标签-值的字典形式存入args字典
# 默认已经添加了--help和-h
parser.add_argument('--verbose', '-v', action='store_true', help='verbose mode')
parser.add_argument('--add', '-a', action='append', help='add range or single value')
parser.add_argument('--remove', '-r', action='count', help='remove range or single value')



args = parser.parse_args()                                                        
if args.verbose:
    print ("Verbose mode on!")
else:
    print ("Verbose mode off!")
```


```bash
$ ./main -a 1 -a 2 -a 3 -r -r -r
input argv is ['./main', '-a', '1', '-a', '2', '-a', '3', '-r', '-r', '-r']
Verbose mode off!
['1', '2', '3']
3
```