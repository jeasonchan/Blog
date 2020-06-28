@[TOC]
# 1 IOC容器
IOC容器就是负责实现IOC这种设计模式的容器，具体的作用就是对资源进行控制，负责资源的创建、销毁，程序本身从IOC容器中取用资源，实现了资源控制权的反转，资源控制权不再在程序自己手中，而是在IOC容器中。
目前，本人学习到的IOC容器有两类，一种是BeanFactory接口，另一种是ApplicationContext 接口，这两个接口下面还另外有各自的实现类，实现xml文件读取bena配置或者从java类中读取bean配置。
# 2 BeanFactory
最简单的IOC容器，没啥高级功能，在我使用的spring4中，它的的一些实现类已经被标记为过时……比如xmlBeanFactory，可能是多出来其他更优的实现类。
在资源宝贵的移动设备或者基于 applet 的应用当中， BeanFactory 会被优先选择。否则，一般使用的是 ApplicationContext，除非你有更好的理由选择 BeanFactory。
# 3 ApplicationContext 
它是BeanFactory的子接口，在BeanFactory的基础上新增了很多高级功能，是spring中较为高级的容器。
和 BeanFactory 类似，基本功能就是，加载配置文件中定义的 bean，将所有的 bean 集中在一起，当有请求的时候分配 bean（分配bean这一过程就是资源控制权的反转）。 另外，它增加了企业所需要的功能，比如，从属性文件中解析文本信息和将事件传递给所指定的监听器。
ApplicationContext 常用的接口实现有：
* FileSystemXmlApplicationContext：该容器从 XML 文件中加载已被定义的 bean。在这里，你需要提供给构造器 XML 文件在操作系统中的绝对路径。这个路径可以主函数的参数传递进去，实现灵活装配。
* ClassPathXmlApplicationContext：该容器从 XML 文件中加载已被定义的 bean。在这里，你不需要提供 XML 文件的完整路径，只需正确配置 CLASSPATH 环境变量即可，因为，容器会从 CLASSPATH 中搜索 bean 配置文件。
* WebXmlApplicationContext：该容器会在一个 web 应用程序的范围内加载在 XML 文件中已被定义的 bean。
* AnnotationConfigApplicationContext：提供某个类即可，容器会根据这个类中的注解和信息，对bean进行组装



