# 0 背景
调用可执行文件，比如，调用ls、cd、mkdir等命令

参考文档：
在python中实现调用可执行文件.exe的3种方法  https://www.jb51.net/article/164762.htm

python调用外部可执行文件的三种方法   https://blog.csdn.net/qq_40268672/article/details/109537319


# 1 代码实践
```py
result = subprocess.call("cat /home/jeason/projects/pythonStudy/MainForFirstCommit.py",shell=True)
print(result)
"""
输出：
# encoding=utf-8

if __name__ == "__main__":
    print("hello world")
0

最后还带了一个0，表示主函数return 0这个返回值

"""

statusCode,output=subprocess.getstatusoutput("cat /home/jeason/projects/pythonStudy/MainForFirstCommit.py")
print("statusCode:{}".format(statusCode))
print(output)
"""
statusCode:0
# encoding=utf-8

if __name__ == "__main__":
    print("hello world")
"""


result=os.system("cat /home/jeason/projects/pythonStudy/MainForFirstCommit.py")
print(result)
"""
输出：
# encoding=utf-8

if __name__ == "__main__":
    print("hello world")
0

最后还带了一个0，表示主函数return 0这个返回值

"""


# f 是一个对象
f = os.popen("cat /home/jeason/projects/pythonStudy/MainForFirstCommit.py")
print(f)
data = f.readlines()
f.close()
print(data)
"""
从f对象中readlines读取程序的标准输出，形成数组
['# encoding=utf-8\n', '\n', 'if __name__ == "__main__":\n', '    print("hello world")\n']
"""
```

综合看，还是statusCode,output=subprocess.getstatusoutput()比较方便，能先根据状态码判断一下程序是否成功执行。