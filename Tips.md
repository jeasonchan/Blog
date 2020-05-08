
# Tips
1. 代码修改后，多进行格式化操作。
2. 命名要规范：包名全小写，类名第一个字母大写。方法名以动词开头，采用动词加名词的格式；源代码包括的注释，尽量不要出现中文，防止出现乱码；
3. 圈复杂度不能太高
4. 类耦合度要尽量松散，便与拓展和迭代
5. 具体的测试用例不写到测试代码中，要使用文件或者其他加载的形式，减少测试代码量
有些变量/方法可以采用内联的形式，减少部分变量创建的开支（C++编译时好像可以自动优化这部分变量？？Java是否可以？？）
6. 一般情况下，源码文件夹src下面分为main和test文件夹，main下的java文件夹才是真正源码文件夹。Idea导入项目时，要标记Sources和Tests文件夹，否则无法进行代码提示不全功能。一般对应的时main和test文件夹。
7. 每次新建或者编辑文件时注意当前的编码模式，GBK. UTF-8之类的
8. 第一个src的上一层作为project
9. 追加commit命令
git add /CODE/csp/setup/src/java/setup/common/ViewControl.java
git commit --amend
git review -R -v
10. traceback中的source和sink，source是问题的最原始源头，sink是直接抛出出问题的地方，Consle中的错误行数是错误发生时执行到的行数。
11. 进行强制类型转换时先进非null检查，进行方法调用. 成员变量调用时先进行非null检查。常见的动作前检查有clone. copy，以及Map. string. collection的初始null。
12. 流，一定要有try finally结构，并在finally中集中关闭流，绝对避免随时随地关闭流；并且，新流已旧流为构造参数时，一定要按照先关外层流（新流），再关里层流（旧流）的顺序。
13. gti项目时，必须进行基本的项目概况检查，比如，编码格式（GBK. UTF-8）. 版本分支等。项目的入口文件夹一般是src的上一层文件夹，入口文件夹选错会无法读取idea的项目结构文件，类的相互引用出现问题。
14. 改coverity和kw，空指针检查一定要避免检查过早导致抛出异常而中断整个流程，比如，尽量在变量使用的上一行进行检查。一定要避免希望始终执行的语句放在可能抛异常的后面，防止主线程跳出，整个程序崩溃。
15. commit之前一定要检查修改过的地方，防止IDE自动引用一些包
16. git add  .   将更改添加到缓冲中，准备提交，可add多个文件；然后，git commit -m ”say sth”；提交之前，通过log，并结合status确认一下别人是否有改动自己自己准备提交的文件。
17. try catch finally 结构中，finally中只尽量少写代码，只进行必要的资源释放操作
18. idea中 Ctrl +I 直接实现接口方法；Alt+Insert 列出要重写或者要实现的方法
19. 项目的目录结构（common为常用的. 可以公用的东西，kafka单独一个包）；
20. 没有需求方法/函数暂时不写，用到的时候再加上；
21. 多了解使用lombok等常用的注解，能减少代码量和提高单元测试的覆盖率，比如，@getter,@setter；
22. 需要后续处理汇总或者要感知操作的结果时才提供返回值，其余情况下，错误. 异常等23. 信息处理直接在执行的过程中进行检查并抛出，不必暴露给调用者；
23. 命名习惯问题，比如RncKeeper和RncQueue，驼峰式，动词+名词；
24. 变量声明，用大接口/父类声明，实例化时再用具体类向上转型，大声明，小实例化，如，Map<String, Object> map=new hashmap<>();
25. 思考新类和旧类的关系，Has-A关系采用注入，Is-A关系采用继承;  
26. 注意：项目jdk版本（project structure）、项目语言等级（project structure）、模块语言等级（project structure）、模块/项目编译等级（java compiler）;可以手动设置，也可以通过maven进行控制。
27. 偏底层的微服务，不需要对传入参数进行安全校验，保证自己的微服务在能在错误参数中不崩溃即可。
28. 在方法/类前（/**+回车），生成方法/类文档，（/*+回车）生成的普通的多行注释，(//)生成单行普通注释，（ctrl+/）直接注释掉当前一整行内容。
29. java里面只有基础数据类型是值，数据容器（数组、列表、类什么的）的名称全是引用/指针，无论换多少名字，只要是引用/指针，指向的内存区域都是一样的，因此，只要map put进的是数据容器的指针/引用，先放进map后更新数据和先更新数据后放进map，完全没有影响！将引用作为方法的入参，并且在方法内部对引用所指向的内存区域进行修改，会改变的原始值，除非进行深拷贝（可使用字节流和对象流进行深拷贝）。
30. 只有使用迭代器对象的remove才能删除迭代器指向的实际对象，且不会停止继续迭代， 达到了删除元素时，不破坏遍历过程的目的。Iterator实例是迭代器实例，Iterator实例的next()方法返回下一个真实对象的引用。
31. 原来的线程并不会等待fork出来的线程执行完毕再向后继续进行，除非join一下，原线程会等fork线程执行完 再继续进行join后面的语句。
32. 写微服务时，一般写两个ym配置文件，放在resources文件里的yml文件作为本地启动的使用的，放在配置文件夹里的，是bin中的启动脚本的使用，就是真正上线时使用的yml文件，同时，main函数也要变一变，一是要适应本地启动时自动调用的本地启动的yml文件，二是适应使用脚本调用时接收外部给定的配置文件夹中的yml文件，典型的写法如下：
```java
private static final String CONFIGURATION_FILE = "server.yml";
public static void main(String[] args) {
        try {
            if (ArrayUtils.isNotEmpty(args)) {
                new App().run(args);
            } else {
                URL url = App.class.getClassLoader().getResource(CONFIGURATION_FILE);
                if (url != null) {
                    String localConfigFile = url.getFile();
                    args = new String[]{"server", localConfigFile};
                    new App().run(args);
                }
            }
        } catch (Exception e) {
            log.error(e.getMessage(), e);
        }
    }
```
App.run()接收并解析到yml中的键值对，尽管App.run（Config config，Env env）的入参是两个类实例，但是！yml中的键值对能自动序列化为config和env的成员变量值，从而达到将类进行实例化，传入的还是两个类实例，这一过程只是减少了手工读取值，再用读到的值去初始化入参实例的过程。
33. 微服务里面，目前常用的基于slf4j的日志记录方法有两种：
```java
//第一种，基于lombok的注解
import lombol.extern.slf4j
@slf4j
class Abc{	
	public void function(){
		log.info("{}大于{}","123","111");
	}
}
//第二种，基于slf4j原生
import xxxx.xxxx.slf4j
class Abc{	
	private static final Logger LOGGER=LoggerFactory.getLogger(Abc.class);
	public void function(){
		LOGGER.info("{}大于{}","123","111");
	}
}
```
34. idea中，放在main——resources中，编译的时候，文件会自动编译到getClassloader()路径中，再结合getResource()实现先对文件URL的获取，从而实现对文件的访问。
35. try  catch 执行顺序，被catch之后，继续顺序执行，比如
```java
        try {
            //成功流程
            //成功流程写在一起符合一般人的思维习惯
            //但是！！！如果正常流程很长很长，下面的异常后的流程很短
            //还是可以考虑将异常流程的return写在catch中的
        } catch (IOException e) {
            //异常处理
        }finally{
        	//不管是成功还是异常，都要处理的事情，比如关闭流
        }
			//异常出现后要走的流程
```
正常流程在try当中，发生异常，并且异常被捕捉到catch中，执行完catch会继续执行后面的，catch花括号后面的是异常流程！新收获！并且try块中有return语句时，仍然会首先执行finally块中的语句，然后方法再返回。连续多个catch是，只要有一个catch生效了，后面的catch全都“消失”。

36. 使用File对象创建文件时，必须要要先创建其父文件夹路径，再创建对象。创建父文件夹路径时，还要先判断该路径存不存在，不存在时，再才要创建父文件夹路径。

37. catch return finally执行顺序及返回值问题：

结论：https://zhuanlan.zhihu.com/p/133667530

* 在catch中有return的情况下,finally中的内容还是会执行，并且是先执行finally再return。
* 需要注意的是，如果返**回的是一个基本数据类型**，则finally中的内容对返回的值没有影响。
因为返回的是 finally执行之前生成的一个副本。
* 当catch和finally都有return时，return的是finally的值。（毕竟finally先执行）

