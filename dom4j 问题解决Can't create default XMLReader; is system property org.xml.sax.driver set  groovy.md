@[TOC]
# 1问题背景
想写一个脚本或者程序来更新工作流中的脚本内容，来避免每次都要去手动赋值粘贴的麻烦，由于python的文本操作不熟悉，又设计到xml文件的读写，于是考虑采用groovy“脚本”的方式，进行xml文件的读取以及其中内容的修改，直接使用dom4j还是groovy.util里自带的xmlpaser类，总时在第一就报错，比如：
```java
SAXReader saxReader = new SAXReader()
```
错误类型就是标题的“Can't create default XMLReader; is system property org.xml.sax.driver set  groovy”，意思是，我没有设置xml的解析“引擎”，但是！同样的使用dom4j的java代码，在项目运行时是完全没有问题，然后我就用junit写了个UT，逻辑就是解析xml文件，UT是能能对xml文件进行解析的，这就很奇怪了，对以上情况进行原因分析和提出解决方法，使在groovy中能够使用dom4j。
# 2原因分析及解决方法
## 2.1指定xml解析“引擎”
```java
System.setProperty("org.xml.sax.driver","javax.xml.parsers.SAXParser")
```
通过以上语句，报错的内容变了，就是找不到后面的SAXParser类。
于是，尝试将此类进行import，还是报无法找到此类……
推测，应该是class path什么的，导致无法访问到这个类……原因不明
## 2.2显式增加xml解析的jar包
可能是dom4j中自带的解析引擎无法访问，导致包解析引擎没设置的错误，于是引入别的独立的解析引擎，来时groovy脚本中dom4j使用正常，使用maven在java项目中额外导入依赖/lib：
```xml
        <dependency>
            <groupId>apache-xerces</groupId>
            <artifactId>xml-apis</artifactId>
            <version>2.7.0</version>
        </dependency>
        <dependency>
            <groupId>apache-xerces</groupId>
            <artifactId>xercesImpl</artifactId>
            <version>2.9.0</version>
        </dependency>
```
然后，像往常一样进行xml解析：
```java
File xmlFile=new File("test.xml")
SAXReader saxReader = new SAXReader()
Document document = saxReader.read(xmlFile)
Element rootElement = document.getRootElement()
//statements
```
**问题解决！**
在groovy脚本中使用dom4j需要额外，导入独立的xml解析引擎xml-apis和xercesImpl。
