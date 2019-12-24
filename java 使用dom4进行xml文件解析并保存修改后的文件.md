@[TOC]
# 0原理浅析
百度百科：
dom4j是一个Java的XML API，是jdom（java原生的文件对象模型类库）的升级品，用来读写XML文件的。dom4j是一个十分优秀的JavaXML API，具有性能优异、功能强大和极其易使用的特点，它的性能超过sun公司官方的dom技术，同时它也是一个开放源代码的软件，可以在SourceForge上找到它。

个人理解：
dom4j将xml文件解析到内存中后形成文件模型树，即为一个Document实例，然后该Document实例就和那个xml文件没有关系了，对Document实例的修改并不会影响xml文件，需要保存/写入到本地文件中，要么覆盖源文件要么存为新文件。

重要组成部分：
* 标签，也就是元素，\<xxxxxx\>，这就是一个标签
* 属性，\<xxxxxx   id="start" name="start" \>，其中的id、name就是标签的属性
* 内容， 对于非自闭和标签，两个标签之间的内容就是这个标签的内容，比如：\<xxxxxx\> 哈哈哈哈 \</xxxxxx\>
# 1文件解析
```java
File xmlFile = new File("D:/test/test.xml");
SAXReader saxReader = new SAXReader();
Document document = saxReader.read(xmlFile);
//得到当个根元素
Element rootElement = document.getRootElement();
//得到根元素中，名为“process”的元素
Element process_element = rootElement.element("process");
//得到该元素内，所有元素的列表
List<Element> scriptTaskList = process_element.elements("scriptTask");
for (Element script_element : scriptTaskList) {
	//得到该元素，属性id对应的值，也可以先得到其名字为id的属性对象实例，再取其值
    String groovyName = script_element.attributeValue("id");
    //得到标签间的内容，都会转变为文本
	println script_element.element("script").getText();
	//xml中的内容可能是文本，也可能是可直接执行的脚本，为了避免xml自己就去解析内容，
	//使用CDATA（character data）来包裹内容，让xml自己不要去解析
    DefaultCDATA cdata=new DefaultCDATA("23333");
    //set操作会直接覆盖原先的内容，set的内容是List
    script_element.element("script").setContent([cdata]);
}
```

# 2文件保存
```java
OutputFormat outputFormat = OutputFormat.createPrettyPrint();
outputFormat.setEncoding("utf-8");
//注意！！！！一定要用"\\."表示点号；取文件的除去文件后缀的部分，以便另存为新名字
File xmlFile2 = new File(xmlFile.getName().split("\\.")[0] + "_"+System.currentTimeSeconds() + ".xml");
//将xmlWriter和输出文件关联
XMLWriter xmlWriter = new XMLWriter(new FileWriter(xmlFile2), outputFormat);
//将修改过后的Document实例对象，通过流，写入到文件中
xmlWriter.write(document);
//关闭流；其实还少关了之前new出来的FileWriter流……生产代码一定要关闭！
xmlWriter.close();
```
# 3拓展
<https://baike.baidu.com/item/dom4j/828750>
递归操作什么的，参考百度百科中的介绍。
