# 1 变量类型
![变量类型](https://img-blog.csdnimg.cn/20191212085226730.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2plYXNvbl9jaGFuX3pqdQ==,size_16,color_FFFFFF,t_70)
定义变量：
$:定义scalar变量，\${var}；
@定义list变量，@{var}；
&定义dict变量，&{var}，此种类型只在新版本ride中支持。

使用变量：
$还用来取值，包含scalar, list和dict变量，其实就是取\${expression}当中expression这个表达式的值，和typescript中  console.log(`我的年龄是${person.age}`)  中的用法十分相似，同时也和bash中的取值\${var}十分相似；RF中的常用方法，比如:
\${200}，取200这个数值，因为一般情况下，只写200，RF只认为是字符串；
\${name=="jeason"}，值为$\{True}或者\${False}，布尔值；
\${list[0]+list[1]}，返回数组/列表的索引值的计算结果；

# 2 详细的使用举例
RF的语法，就是一种表格语言，一个表格里放一个变量/运算符号，不像别的语言一样，用空格进行间隔。

# 2.1 scalar
https://www.cnblogs.com/Mr-ZY/p/11697077.html
定义：



rf语法：主要是断言和关键字调用时的输入输出
https://www.cnblogs.com/ww-xiaowei/p/10338952.html
https://www.cnblogs.com/ww-xiaowei/p/10330733.html

