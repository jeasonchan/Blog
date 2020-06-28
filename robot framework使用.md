@[TOC]
# 概念
* 关键字：keywords，很多测试python脚本的合集，执行一个关键字时，就会执行这些python脚本。传入关键字的参数，通过字符匹配，分别传给每个python脚本。
* 关键字感觉就像一个函数，测试case调用关键字，并给关键字传参和处理关键字的返回值
* 

# 安装
参考IBM开发者社区<https://www.ibm.com/developerworks/cn/opensource/os-cn-robot-framework/index.html>
1. 安装Python 2.7，勾选add to path；Python 3的兼容性对于这个框架有问题
2. pip install robotframework
3. pip install wxPython；安装依赖的图形界面库；
4. pip install robotframework-ride

在powershell中测试运行，会出现界面：
```powershell
cd D:\Python27\Scripts
python ride.py
```
或者写shell脚本，打开方式选择bash，脚本内容和powershell相同。
# 使用步骤
1、关键字目录下，new resource，用来容纳一系列关键字，是robot后缀
2、给robot文件导入library，指向写好的py文件，py文件名一定要和class名相同
3、右键robot，新建关键字，做好关键字命名工作
4、设置关键字的输入参数（arguments）和输出参数（return value），输入参数是给下面python函数传参用的，输出参数可以用来接收python函数的返回值，并返回给关键字的调用者
5、在下方设置要使用的Python函数的名字，设置python要用到的关键字的输入参数，和接收函数返回值的关键字返回值，能用到的参数只能是上面关键字定义好的
6、创建测试suite，file，txt
7、导入上面创建的关键字合集，即robot文件
 8、右键测试suite，新建测试case，即为测试用例
 9、直接输入关键字名称，对关键字进行调用，并且输入参数，输入参数的个数一定要大于等于关键字所需要的数量，输出参数按需赋值
 10、对测试case打勾，运行查看结果
