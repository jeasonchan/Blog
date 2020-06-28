# 1 背景
不论是什么的编程语言，本质上都是基于文本内的内容，编译器/执行器执行相关逻辑，文本内容就是代码，编译器/执行器运行的环境就是执行环境。

因此，代码本身的编码类型不同会造成乱码现象，同时，由于操作性系统的默认编码类型不同也会造成编码现象。

代码本身乱码，很常见。比如，原本是utf-8的代码文档，文档中含有中文，但是，别人打开时，如果使用gbk编码打开，中文部分会乱码。
操作系统默认编码带来的乱码，有时候也存在，比如，读取文件路径，java默认是utf-8编码，Linux发行版也是默认utf-8编码，直接读取路径，并打印出来，路径中的中文能正常显示。但是，window中，目前的中文默认是gbk编码，直接读取过来的中文也是gbk，再decode为utf-8的形式给人类展示。
	
# 2 字符的本质和表示
字符就和其他变量一样，在代码/文本里输入后，会被转成字节进行存储，以字符串为例，英文字符、中文字符赋值给变量或者存储时，也要先转换成字节，当代码文件是以utf-8编码时，所有的字符会按照utf-8编码规则转为字节，使用时再取出字节，再按照utf-8的规则转换为原先可读的字符。
在网络传输或者从外部读取东西时，程序直接取到的string类型的变量/对象，其实已经偷偷做了一步：按照程序默认的规则将字节转换为字符，对于java，默认的规则是utf-8；对于python3，默认的是utf-8；对于python2，默认的是Unicode；
因此，如果我们的是事先已经知道我们从外部读到的字符和我们默认的字符编码规则不同，我们的必须先将得到字符，先按照原始编码规则encode为字节，再按照我们默认/目标的转码规则进行decode，最终得到正确、可用的字符。

# 3 以python为例实践字符相关操作
```python
# coding=utf-8
import os
import sys

if __name__ == '__main__':
    path: str = "哈哈哈哈"
    path2: str = "嘿嘿嘿".encode("utf8").decode("utf8")
    print(path, path2)

    print("\n=============================")
    print("系统默认编码是:", sys.getdefaultencoding(), "\n")

    filelist: list = os.listdir("C:\\CRroot\\documents\\")  # 此处返回的list中的中文是以GBK编码的，你可以通过查看cmd窗口属性看到。
    for path in filelist:
        # if os.path.isdir(path):
        print(path, os.path.isfile(path), type(path))

    fileStream = open(".\\GBK编码的中文文档.txt", "rb")
    print(fileStream.read())  # 输出是 b'\xd2\xbb\xbe\xe4\xd6\xd0\xce\xc4\na english\n'

    fileStream = open(".\\GBK编码的中文文档.txt", "rb")
    try:
        print(fileStream.read().decode("utf8"))
        # 输出是 'utf-8' codec can't decode byte 0xbe in position 2: invalid
        # start byte <class 'UnicodeDecodeError'>
    except Exception as message:
        print(message, type(message))

    fileStream = open(".\\GBK编码的中文文档.txt", "rb")
    print(fileStream.read().decode("gbk"))  # 输出是的是正确的GBK文档中的中文内容'
```
输出是：
```bash
C:\XXXXX\Python36\python.exe C:/XXX/documents/codeproject/pythonStudy/python编码问题/Main.py
哈哈哈哈 嘿嘿嘿

=============================
系统默认编码是: utf-8 

XXX.mv.db False <class 'str'>
XXX.trace.db False <class 'str'>
codeproject False <class 'str'>
Database1.accdb False <class 'str'>
Database2.accdb False <class 'str'>
Default.rdp False <class 'str'>
desktop.ini False <class 'str'>
MobaXterm False <class 'str'>
My Kindle Content False <class 'str'>
Onedrive False <class 'str'>
Tencent Files False <class 'str'>
Visual Studio 2017 False <class 'str'>
WeChat Files False <class 'str'>
【XXXX】杂七杂八 False <class 'str'>
【XXXX】杂七杂八 False <class 'str'>
腾讯影视库 False <class 'str'>
自定义 Office 模板 False <class 'str'>
b'\xd2\xbb\xbe\xe4\xd6\xd0\xce\xc4\na english\n'
'utf-8' codec can't decode byte 0xbe in position 2: invalid start byte <class 'UnicodeDecodeError'>
一句中文
a english
```

再来一个史上最简单易懂的例子：
```python
 	s: str = "hahaah啊啊哈哈"
    print(type(s), s, len(s))
    s_unicode = u"hahaah啊啊哈哈" # 和上面的一样，只是显式声明，是一个unicode编码的字符穿
    print(type(s_unicode), s_unicode, len(s_unicode))

    s_bytes: bytes = s_unicode.encode()  # 字节串
    print(s_unicode.encode())
    print(s_bytes.decode("gbk"))
    print(s_bytes.decode("utf8"))
```
输出：
```
<class 'str'> hahaah啊啊哈哈 10
<class 'str'> hahaah啊啊哈哈 10
b'hahaah\xe5\x95\x8a\xe5\x95\x8a\xe5\x93\x88\xe5\x93\x88'
hahaah鍟婂晩鍝堝搱
hahaah啊啊哈哈
```	
用错误的编码类型GBK，去编码字节串，果然是乱码了。
