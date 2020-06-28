# 基础
在java工程中获取文件的方式有多重，目前见到的有以下几种：
* 直接使用文件的绝对路径，在idea中copy path即可，但是，缺点是，项目代码换个机器可能就找不到那个文件。
* 使用"src//aaa//bbb//ccc//file.txt"的方式，不打包的话，任何机器在运行时都可以获取到这个文件，缺点是，无法将文件打包进一整个包里，不打包的项目可忽略这个缺点。
* 和target文件夹里编译生成的 .class文件放在一起，然后使用A.class.getResources("xxxxx")的方式获取到文件URI，几乎毫无缺点，就是git同步的时候要选择同步target文件夹。

因此，推荐后两种方式获取文件！第二种方式较为简单直白，现对第三种方式中的要点进行分析。
假设target文件中编译生成了这样的文件结构：target——calss——com——jeason、file6666.txt——A.class、file.txt），即，file1.txt和jeason目录同一级，file.txt和A.class文件同一级，位于jeason文件夹中。分别获这两个文件可以通过以下方式：
```java
URl url=A.class.getResource("file.txt");//正确，使用了相对路径
URl url=A.class.getResource("/com/jeason/file.txt");//正确，使用了绝对路径
URl url=A.class.getClassLoader().getResource("com/jeason/file.txt");//正确，使用了相对路径
URl url=A.class.getClassLoader().getResource("./com/jeason/file.txt");//正确，使用了绝对路径
URl url=A.class.getResource("../file6666.txt");//正确，使用了相对路径，从父级目录取文件
//用了getClassLoader()的绝对不能以"/"开头，因为本身就已经有了"/"，至少也应该使用"./"表示根目录，即com目录的上一层目录
```
# 补充
* 据说，有时候在windows下使用getResourece()获取路径是正常，但是linux将URI转为字符串时经常出现感叹号的空格，导致文件读取还是失败，因此，可以考虑将getResource函数替换为"getResourceAsStream()"，即为：
```java
URl url=A.class.getResourceAsStream("../file6666.txt");
```
* 使用IDEA时，放在src resources中且被标记为resources的文件夹中的文件，项目编译后，这些文件会被自动放到getClass().getClassLoader()下面
* 从将url对象getFile（）时，得到的文件路径经常可能乱码，使用URLDecoder.decode(configFile,"utf-8")方法确保得到的文件了路径是utf-8编码的

