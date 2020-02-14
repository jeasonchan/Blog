# 0 声明

本文并非原创：

* 1.1~1.4参考了<https://legacy.gitbook.com/book/yulinfeng/-java/details>
* 1.5 参考了<https://blog.csdn.net/ZixiangLi/article/details/82853950>
* 

感谢原作者。

# 1 创建型设计模式

创建型的设计模式一共有5种，顾名思义，用于创建对象实例的一种设计模式。由于**直接new对象会造成耦合度高、扩展性不强等问题**，遵循创建型的设计模式进行编码设计能较好的避免这种问题的发生。使得代码更加健壮，更具有扩展性和更低的耦合度。

## 1.1 工厂模式

工厂，是用于生产商品的地方。现实生活中，一个人并不会自己生产铅笔、钢笔、毛笔，它会直接选择从工厂生产出来商品。

在面向对象的世界里，我们习惯于将各种事物进行抽象，将各种各样的对象进行组合以便实现我们的业务逻辑。而在组合过程中，我们需要自己来创建对象，**工厂模式定义一个用于创建对象的接口，让子类决定实例化哪一个（不同的子类都实现创建对象的接口方法，最后都调用接口的方法即可）**。这也是它是创建型设计模式的原因，用于创建同一类型的对象，该种对象可能是继承同一接口或者抽象类。

### 1.1.1 原型介绍

直接先用UML类图来直观感受下工厂模式（**非简单工厂模式**）。

![工厂模式UML类图](https://patterns.coderbuff.com/chapter1/factory_design_pattern/factory_design_pattern.png)

对于工厂方法，首先是需要定义一个用于产生具体对象的工厂接口（IFactory接口类，createFactory()方法），其次还有一个具体对象的公共父接口（IProduct接口） 或者 抽象类（上图使用的是接口，暂无抽象类）。产生具体对象是工厂接口的子类（上图中的ConcreateFactory），注意是工厂接口的子类。如果只定义一个实体工厂类用于生成类，就是简单工厂模式了。

在不了解工厂模式的情况下看到这个UML几乎是处于茫然状态，下面从不借助工厂模式到借助工厂模式的代码演进进行了解。

**阶段1——强耦合，结构化**

```java
ConcreteProduct product = new ConcreteProduct();
```

在这个阶段并没有理解到面向对象编程的含义，代码的编写还是沿用结构化编程的方式，如果此时有**另一个相似的类**也要产生具体对象，则继续是单独写一个类而不会对两个类进行抽象。

```java
ConcreteProduct product = new ConcreteProduct();
ConcreteProduct2 product2 = new ConcreteProduct2();
```

**阶段2——强耦合，OOP**

此时会对两个相似的、具体的产品的公共部分进行抽象为一个接口——Product。代码的编写演变到如下所示。

```java
Product product = new ConcreteProduct();
Product product2 = new ConcreteProduct2();
```

进行到这一阶段，你已经知道了面向对象的抽象威力，这能使你省去大量代码。

**阶段3——松耦合，OOP（简单工厂模式）**

上一个阶段我们利用了OOP的思想，但还是未能消除**强耦合**带来的问题，尽管直接new一个对象是很常见的一个动作，但在现如今的编码中，将控制权交由第三方来控制（IoC）能让代码耦合度更低，**高内聚低耦合**是软件编码过程中的黄金定律，耦合度更低的代码具有复用性更强，更容易测试，扩展性更好的特点。

```java
Product product = Factory.createProduct('concreteProduct');
Product product2 = Factory.createProduct('concreteProduct2');
```

在这段代码中使用的实际上是**简单工厂模式**，这和**工厂方法模式**有细微的区别，这里通过传入一个参数创建一个特定的实例对象（ConcreteProduct），这就是一种IoC思想——将创建对象（ConcreteProduct）的控制权交给了第三方（Factory），从而实现了对代码的解耦。

```java
/**
 * 简单工厂模式
 */
public class Factory {
    public static Product createProduct(String product) {
        switch (product) {
            case "concreteProduct":
                return new ConcreteProduct();
            case "concreteProduct2":
                return new ConcreteProduct2();
            default:
                return null;
        }
    } 
}
```

进行到这一步，我们代码已经改造得近乎完美，看似解决了耦合性问题，又利用OOP的思想。工厂模式似乎还并未派上用场，接下来问题马上就来。

此时，我们因为一个新的需求需要再创建一个新的Product实现类，首先当然我们能直接实现Product类——ConcreteProduct3，由于OOP的思想我们能直接复用。接下来要创建一个ConcreteProduct3对象，**我们需要在原有的工厂类增加一个case判断语句**。

```java
/**
 * 简单工厂模式，新增case判断语句
 */
public class Factory {
    public static Product createProduct(String product) {
        switch (product) {
            case "concreteProduct":
                return new ConcreteProduct();
            case "concreteProduct2":
                return new ConcreteProduct2();
            case "concreteProcduct3":
                return new ConcreteProduct3();
            default:
                return null;
        }
    }
}
```

再将传入方法的参数修改为"concreteProduct3"就能获取ConcreteProduct3实例对象。

注意上面的关键——需要在原有的工厂类上**修改代码**，**代码扩展性的体现并不是修改原有代码来新增功能，而是新增代码就能扩展一个新功能。**显然，上面的简单工厂模式没有满足这一点要求，因为它**动了原有代码**。

这个时候就该本文的主题——工厂模式登场了。

工厂模式也有一个公共的工厂接口，用来产生工厂对象，要新增功能，只需要新增一个工厂类并实现该工厂接口，就能拓展出新功能。

```java
/**
 * 工厂模式，工厂接口
 */
public interface IFactory {
    IProduct createProduct();
}
```

此时并没有唯一的工厂类，而是需要创建各个具体对象实例相应的工厂，例如ConcreateProductFactory。

```java
/**
 * 工厂模式，生成具体的工厂类
 */
public class ConcreateProductFactory implements IFactory {
    public IProduct createProduct() {
        new Product();//该类实现了IProduct接口
    }
}
```

最初的`Product product = new Product()`就演变成了如下代码。

```java
IFactory factory = new ConcreateProductFactory();
IProduct product = factory.createProduct();
```

同样，现在新增了一个具体对象，此时我们不必修改原有代码，只需要一个实现IFactory接口的工厂类ConcreateProduct2Factory，返回IProduct接口的实现类的实例即可。

```java
IFactory factory = new ConcreateProductFactory();
IProduct product = factory.createProduct();
//新增一个功能类
IFactory factory = new ConcreateProduct2Factory();
IProduct product2 = factory.createProduct();

```

这是工厂模式和简单工厂模式最大的区别——**不必在原有代码上进行修改，而是通过新增的方法扩展新的功能。**

讲到这里，或许你有一个疑问在阶段2的时候你说直接new一个对象这会造成强耦合，所以在阶段3使用了IoC思想将创建对象的控制权交给了第三方，但是**到最后的工厂模式`IFactory factory = new ConcreteProductFactory()`不也直接new了一个工厂实例对象**吗？这不也产生了强耦合吗？这和直接`new ConcreateProduct()`到底有什么好处呢？并且好像只看到了代码的复杂度增高了，实际上也并未消除强耦合。

这里实际上产生了一种错觉，这个例子没有实际业务场景无法体会工厂设计模式所带来的优越感，当然它有缺点，**它的缺点就是代码复杂度增加了，并且需要和具体类所对应的工厂强耦合**。

之所以造成一种错觉工厂设计模式不如直接new来得快，来得爽，正是因为这里所有的代码都是你一人写的，然而在实际开发过程中情况更为复杂并且通常是多人合作，多人合作提供给对方的往往是一个接口。也许IProduct接口是你写的，而具体实现类Product1、Product2则是由他人写的，此时在其他人没有完成代码编写时你无法用Product1、Product2创建实例对象，然而利用工厂模式他人却能给你提供一个工厂返回一个接口。

**在实际编码中，往往来得快来得爽的代码大多都会埋下未知的隐患。**



总结一下，工厂模式就是用Factory接口的实现类，去产生Product接口的实现类。

### 1.1.2 Java源码中的工厂模式

在Java源码中Iterator迭代器的实例对象创建使用的就是工厂模式，所以这里举例Iterator探寻JDK作者们是如何将工厂模式应用到Java源码中的。

由于Java源码的多样性和复杂性，有的看起来并不那么像一个“标准”的设计模式，所以原作者对一些类进行简化，例如省去泛型，或将类的继承关系简化，或在分析源码结构时先将内部类剥离出来等等。

先来一段创建Iterator迭代器的代码。

```java
//创建Iterator迭代器
Collection collection = new ArrayList();
Iterator iterator = collection.iterator();

//工厂模式原型
IFactory factory = new ConcreteFactory();
Product product = factory.createProduct();
```

这和介绍中的工厂模式如出一辙（尽管实际上我们可能并不会这么写，`List list = new ArrayList()`的方式更常用）。迭代器创建的本质就是，由Collection接口的实现类（ArrayList），创建（iterator()方法，工厂类实现的方法）Iterator接口的实现类（ArrayListItr）。

所以我们能依葫芦画瓢照着工厂模式的UML类图画出Iterator工厂模式的类图。

![Iterator工厂模式的类图](https://patterns.coderbuff.com/chapter1/factory_design_pattern/jdk_iterator_factory.png)

事实上在Java源码中并没有一个叫ArrayListItr的实现类，Iterator的实现类在ArrayList内部，也就是说在ArrayList有一个内部类。如果我们将Iterator的实现类单独定义，代码的包结构如下所示。

```
Factory Package
├── Collection Interface
├── ArrayList
│
Product Package
├── Iterator Interface
└── ArrayListItr
```

Java源码则将ArrayListItr在ArrayList内部，包结构如下所示。

```
java.util
├── Collection Interface
├── ArrayList
└── Iterator Interface
```

将它们全部放在java.util下是因为还有其他功能的实现需要这么做。不单独实现Iterator从代码结构上来讲能更加清晰，把**Product的具体实现放到具体工厂中作为一个内部类是一种很好的设计思路，即为，ArrayList中返回的Iterator实例，本质上是ArrayList的内部类**。

## 1.2 静态工厂方法

关于工厂模式有几个比较模糊的概念：**工厂方法模式**、**简单工厂模式**、**抽象工厂模式**、**静态工厂方法**。

本文所述**工厂模式**指的是**工厂方法模式**。**简单工厂模式**并未列入23种设计模式之中，它与**工厂方法模式**的区别上文已做解释。有关**抽象工厂模式**会在下一节介绍。

需要注意的是——**静态工厂方法**。

### 1.2.1 原型介绍

静态工厂方法在*《Effective Java》*中的开篇就大力推崇，它和工厂模式的目的都是创建实例对象。对于静态工厂方法，通常类的自身提供一个静态方法用于实例化（当然也可以是一个工具类例如`Collections`），从而避免使用构造器。例如Java源码中的Boolean包装类和**线程池创建类**。

```java
public static Boolean valueOf(boolean b) {
    return (b ? TRUE : FALSE);
}
```

Boolean自身就提供了一个工厂方法用于返回一个Boolean对象。

对应到前面提到的创建一个具体产品的例子，如果ConcreteProduct提供了一个静态工厂方法，则变为以下形态。

```java
Product product = ConcreteProduct.getInstance();
```

这是一个类自身提供了一个静态方法用于对自身的实例化，它取代了new构造器的方式，而这是与工厂模式利用第三方类来创建对象的不同。取代构造器创建对象实例可以有3个好处：

1. **可以自由的对方法进行命名。**命名是门学问，命的好事半功倍，命得差真的要跪。而构造器没法进行命名。

2. **不必每次都创建一个对象。**这实际上就是熟悉的单例模式了，每次获取对象实例的时候都是同一个。

静态工厂方法并不一定都是由类的自身提供，还有可能是工具类，例如`Collections`。

Collections类中另外还实现了几十种集合供我们使用，但这些集合都不能通过直接new的方式获取，而是在Collections的内部提供了静态工厂方法，不能直接实例化大大减少了API的数量，如果几十种集合全部能直接实例化，各种API各种概念都是庞大的工作量，而使用静态工厂方法通过命名则很好的解决了这一个痛点。

```java
Collections.synchronizedMap(new HashMap<>());
Collections.synchronizedList(new ArrayList<>());
Collections.unmodifiableMap(new HashMap<>());
Collections.unmodifiableList(new ArrayList<>());
······
```

Collections通过静态工厂方法导出集合方式不仅仅在于API数量的减小，这就是静态工厂方法的第三个有点：

3. **它能返回原有返回类型的子类型。**

以`Collections.synchronizedList`举例，其内部方法返回的是List类型，而它的真正子类型则是`SynchronizedList`，这在其Collections作为内部类实现。这样我们不必暴露出SynchronizedList，也就是隐藏了我们真正实现的类，但又可以返回真正的对象，这个意义所在就是实现了基于接口编程。本质就是使用静态方法（synchronizedList()），产生了接口（List接口）的实现类（SynchronizedList，但是，该类的实现是内部类）。

## 1.3 抽象工厂模式

在现实生活中商品有许多品牌，每一个品牌的商品又是有许多零零散散的零件组成。例如电脑是一个商品，它由CPU、内存等组成，同时又有华硕、苹果、联想等品牌生产电脑这种商品。

### 1.3.1 原型介绍

在前文的工厂方法模式中一个工厂接口只会包含一个工厂方法，用于创建对象实例，它的子类具体工厂类负责实现接口中的这个方法。

在抽象工厂模式中，一个工厂接口有多个方法，这些方法返回不同的对象实例，工厂接口的具体子类工厂实现这些方法，这些方法都返回不同类型（不类型体现在，实现了不同的接口或者继承了不同的抽象类）的对象实例。从代码形式上看，抽象工厂模式不过是在工厂中定义了多个方法，而工厂方法模式则只有一个。所以，工厂方法模式就是一种特殊的抽象工厂方法。

所以，抽象工厂模式，本质就是用实现了Factory接口的工厂类，去生成实现了A接口/抽象类的实例、实现了B接口/抽象类的实例……

从引子中的例子就可以得出，“生产电脑”是一个工厂接口，在这个工厂接口中定义了“生产CPU”、“生产内存”方法等。而“生产电脑”的实现类又有“华硕”、“苹果”、“联想”等，它们各自“生产CPU”、“生产内存”的方式又不尽相同（也就是方法实现不同）。这种情况下使用抽象工厂方法就是较好的选择。

综上，如果一个对象中并没有其他对象的组合，只是一个单一且不依赖的对象，可以使用**工厂模式方法**，而如果一个对象中**包含**其他对象，相互依赖，此时则可以选择使用**抽象工厂方法**，抽象工厂模式的类图如下图所示：

![抽象工厂模式的UML图](https://patterns.coderbuff.com/chapter1/abstract_factory_design_pattern/abstract_factory_design_pattern.png)

和之前的工厂模式相比，**本质上唯一的区别就是，工厂接口IFactory包含了多个接口**，因此，工厂接口的实现类ConcreateFactory，要同时实现多个接口，从而最终实现将多个对象组合成一个对象（也就是，只需要一个工厂类比如联想，就能同时生产CPU、内存，~~最终给出一个组装对象~~）。

利用抽象工厂模式设计一下，一开始提到的，各品牌生产电脑配件，并最终组装成电脑的类图：

![电脑生产商生产、组装类图](https://patterns.coderbuff.com/chapter1/abstract_factory_design_pattern/computer_abstract_factory_design_pattern.png)

用代码的形式，尝试生成一个电脑实例：

```java
//Lenovo实现了IComputerFactory接口
IComputerFactory lenove=new Lenovo();
ICpu lenovoCpu=lenovo.createCpu();
IMemory lenovoMemory=lenovo.createMemory();
//假设有增加了一个需求，来了一个Apple制造商
//只需要新增代码即可实现功能拓展
IComputerFactory apple=new Apple();
ICpu appleCpu=apple.createCpu();
IMemory appleMemory=apple.createMemory();
```

代码使用起来其实和工厂模式差不多，本质上还是接口的实现类生成接口的实现类，面向接口编程的思想。

### 1.3.2 Java源码中的抽象工厂模式

在Java源码中通过JDBC可以连接操作不同的数据库，它的设计同样是基于抽象工厂模式。严格来讲，它更属于是工厂模式，至于抽象工厂模式和工厂模式在后面会提到。

对于使用JDBC连接数据库，首先会经历下面两个阶段：加载、连接，但是，不同类型的数据库驱动，加载和连接做的事情是不一样的（也就是，实现不同）。

```java
//加载数据库驱动
Class.forName("com.mysql.jdbc.Driver"); 
//连接相应的数据库
Connection connection = DriverManager.getConnection("url", "username", "password");
```

```DriverManager.getConnection()```是我们在**1.1工厂模式**一节的**Tips**中提到的静态工厂方法，它用于创建并返回一个`Connection`数据连接。在其内部，最终调用了一个私有的重载方法getConnection(String, java.util.Properties, Class)`。在这个私有`getConnection`方法中会遍历已经注册的数据库驱动，也就是会得到我们手动加载的MySQL数据库驱动```Class.forName("com.mysql.jdbc.Driver")```。

```java
private static Connection getConnection(String url, java.util.Properties info, Class<?> caller){
	...略...
    for(DriverInfo aDriver : registeredDrivers)
    ...略...
    Connection con = aDriver.driver.connect(url, info);    
}
```

代码中`aDriver.driver`返回的正是MySQL实现的具体驱动类，也就是一个具体的抽象工厂接口实现类——`com.mysql.jdbc.Driver`，它的抽象工厂接口则是Java定义的——`com.sql.Driver`。

分析到这个地方，我们已经能画出**数据库驱动接口及驱动实现类UML图**。

![数据库抽象接口及驱动类UML图](https://patterns.coderbuff.com/chapter1/abstract_factory_design_pattern/jdk_abstract_factory_design_pattern.png)



驱动的抽象工厂接口```java.sql.Driver```中定义的接口不止connect这一个，此处为了简略，只写了一个出来。本质上，从Driver到Conenction实例创建，就是一个的抽象工厂模式，正是因为设计成抽象工厂模式，数据库厂商新增驱动时变得方便，同时，使用者也是直接调用”通用的“接口。

### 1.3.3 小结

通过对比工厂模式和抽象工厂模式，工厂模式只是抽象工厂模式的一直特殊情况，即为，工厂接口包含的方法数为1的特殊情况，抽象工厂模式的本质就是面向抽象、接口编程，代码的实现就是，用工厂接口的实现类去生成”产品“接口/抽象类 实例的过程，从而去使用”产品“的共同特性，比如，方法、属性。

## 1.4 单例模式

通过new关键字创建一个对象实例是我们在学习Java时的第一课，这背后Java做了哪些操作是我们学习JVM时的第一课。

当使用new创建一个对象实例时，JVM会在**堆区域**分配相应的内存，让这块内存也就是这个实例对象还有引用指向它时它便会一直存在，如果没有引用指向它此时便会对它进行GC（可达性分析）。也就是说我们在不断new一个对象时会带来两个问题：

- 占用堆内存空间。
- GC带来的影响。

很多类在无状态（通过传递参数只做逻辑处理）的情况下重复利用就可以了，并不需要每次都new一个对象，例如在Spring中对于bean对象的创建默认都是singleton单例模式，也就是，每次从IOC中get出来的引用，指向同一块堆内存，修改一个会影响全部的同一ID的bean。

### 1.4.1 原型介绍

单例模式可以按不同维度对其进行分类

- 线程安全维度：线程安全的单例模式、线程不安全的单例模式
- 对象创建时机：饿汉式的单例模式、懒汉式的单例模式、singletonHolder式

对于线程不安全的单例模式在多线程环境下对象并不是真正的单例，它还是极有可能被创建为“多例”，所以这种形式的单例模式实际当中并不常见，甚至不会有。接下来的单例模式均是基于**基本线程安全的单例模式**，补充一下，不带synchronized的double check不是线程安全的。

**饿汉式单例模式**，指的是急不可耐的创建一个对象，也就是在类被JVM加载时就会被创建。

```java
/**
 * 饿汉式单例模式
 */
public class Demo {
    private static Demo instance = new Demo();

    private Demo() {}

    public static Demo getInstance() {
        return instance;
    }
}
```

**饱汉式单例模式**指的是不必急着创建对象，而是用到的时候再去创建。

```java
/**
 * 饱汉式单例模式
 */
public class Demo {
    private static Demo instance;

    private Demo() {}

    public static synchronized Demo getInstance() {
        if (instance == null) {
            instance = new Demo();
        }
        return instance;
    }
}
```

但是！由于JVM的指令顺序优化原因，就算加了synchronized关键字，在new一个十分大的对象时，会发生线程安全问题。

**singletonHolder式**，利用JVM内部静态类机制，达到线程安全和效率的平衡。

```java
public class Demo {
    private static class SingetonHoler{
        public final static Demo singleton=new Demo();
    }

    private Demo(){
        System.out.println("调用"+this.getClass()+"构造函数！");
    }

    public static Demo getSingleton(){
        return SingetonHoler.singleton;
    }
}
```

这种方法使用内部类来做到延迟加载对象，因为java机制规定，静态内部类SingletonHolder只有在getSingleton()方法第一次调用的时候才会被加载（实现了lazy，而懒汉模式的单例实例是个实例对象，一load就直接初始化了），而且其加载过程是线程安全的（java自己的实现是线程安全的）。内部类加载的时候实例化一次instance。这种写法最大的美在于，完全使用了Java虚拟机的机制进行同步保证，没有一个同步的关键字。

### 1.4.2 Java源码中的单例模式

Java源码中的单例模式并不常见，Java作为一个基础语言的支撑，将类设置为单例模式显然它是作为一种工具存在。

Java中有一个类在一些特定需求例如性能监控上一定会被用到——Runtime。这个类就是使用了单例模式。

```java
public class Runtime {
    private static Runtime currentRuntime = new Runtime();

    public static Runtime getRuntime() {
        return currentRuntime;
    }
    ······
```

对于获取Java应用程序中的一些状态，并不需要创建很多实例，使用单例模式就是很好的解决方案，Java源码在Runtime类上使用的饿汉式单例模式。

### 1.4.3 Spring源码中的单例模式

前面提到Spring框架中默认产生的bean对象是一个单例，如果想要它是多例可以将bean对象scope属性改为“prototype”表示每次使用这个bean时都会创建一个新的对象实例。另外还有“request”，每一次http request请求都会创建一个新的对象实例。当然还有“session”等。

由于Spring比较复杂，bean对象的创建和初始化过程也会横跨多个类。在这里只摘取出Spring创建单例bean的部分代码，有兴趣可以查看Spring源码。

```java
//AbstractBeanFactory#doGetBean
if (mbd.isSingleton()) {
    sharedInstance = getSingleton(beanName, new ObjectFactory<Object>() {
        
        @Override
        public Object getObject() throws BeansException {
            try {
                return createBean(beanName, mbd, args);
            }
            catch (BeansException ex) {
// Explicitly remove instance from singleton cache: It might have been put there
// eagerly by the creation process, to allow for circular reference resolution.
// Also remove any beans that received a temporary reference to the bean.
                destroySingleton(beanName);
                throw ex;
            }
        }
        
    });
    
    bean = getObjectForBeanInstance(sharedInstance, name, beanName, mbd);
}
```

前文提到过**静态工厂方法**，并且提出其能应用在单例设计模式中。单例模式中，```getSingleton()```方法都被设计成public和static，正是使用了静态工厂方法。

## 	1.5 建造者模式

现实生活中，你购买汽车，却需要知道它是如何组装的这显然不合理。你只需要某个复杂对象，却需要知道它是如何组合的也是不合理的。

将一个复杂对象的构建与表示分离，可以使用同样/相似的构建过程去创建具有共同特征的对象实例。

### 1.5.1 原型介绍

建造者模式原型类图，如下图所示：

![建造者模式原型](https://img-blog.csdn.net/20180926152712885?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1ppeGlhbmdMaQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

通过分析类图，从低级到高级的几个特征类有：

1. 产品类（Product），**个人觉得**，可以是继承了同一接口/抽象类的类，也可以是内部有很多**类组合**的复杂类
2. 具体建造者（ConcreteBuilder），实现的了
3. 建造抽象/接口（Builder），必定包含多个方法，因为建造者模式的目的就是创建复杂的对象
4. 指挥者类（Director）

和之前抽象工厂模式感觉有共同点，此处对比一下异同：

**相同点**：

1. 本质上还是接口的实现类，生成另外的接口的实现类

**不同点**：

1. 建造者模式最终产生的Product只有一个，而抽象工厂模式的产生对象是多个
2. 抽象工厂模式，返回Product对象的最直接方法是抽象工厂接口包含的方法；建造者模式中，返回Product对象的最直接方法是指挥者接口包含的唯一的方法

个人觉得，建造者模式，本质上只是对抽象工厂模式中，抽象工厂的包含的多个接口，做了一个封装，这个封装是可以有顺序的，从而返回**一个**组装好的对象，抽象工厂模式返回的是**散装**的多个对象。

**本质上，还是面向接口/抽象编程，有共性的就抽接口和抽象类。**

### 1.5.2 建造者模式举例

以建造人（Person）为例，因人（Person）的属性不同，分为”胖人“和”熟人“，建造人（Person）的步骤都包括建造头（createHead）、建造身体（createBody）和建造四肢（createLegs）。

产品类，Person类：

```java
@Getter
@Setter
public class Person {
 	public String head;
	public String body;
	public String legs;

	@Override
	public String toString() {
		return this.getHead()+this.getBody()+this.getArms()+this.getLegs();
	}
}	
```

抽象工厂，PersonBuilderFactory：

```java
public interface PersonBuilderFactory {
	
	public abstract void createHead();
	public abstract void createBody();
	public abstract void createLegs();
    
	public abstract Person buildPerson();
}
```

抽象工厂接口的的实现类，胖人制造器（FatPersonBuilder）和瘦人制造器（ThinPersonBuilder）：

```java
public FatPersonBuilder implements PersonBuilderFactory{
    private Person fatPerson = null;
	
	public FatPersonBuilder() {
		if(fatPerson == null){
			fatPerson = new Person();
		}
	}
	
	@Override
	public void createHead() {
		fatPerson.setHead("胖头");
	}
 
	@Override
	public void createBody() {
		fatPerson.setBody("胖身体");
	}
 
	@Override
	public void createLegs() {
		fatPerson.setLegs("胖腿");
	}
 
	@Override
	public Person buildPerson() {
		return fatPerson;
	}

}
```

```java
public ThinPersonBuilder implements PersonBuilderFactory{
    private Person thinPerson = null;
	
	public ThinPersonBuilder() {
		if(thinPerson == null){
			thinPerson = new Person();
		}
	}
	
	@Override
	public void createHead() {
		thinPerson.setHead("瘦头");
	}
 
	@Override
	public void createBody() {
		thinPerson.setBody("瘦身体");
	}
 
	@Override
	public void createLegs() {
		thinPerson.setLegs("瘦腿");
	}
 
	@Override
	public Person buildPerson() {
		return thinPerson;
	}

}
```

人类制造直指挥者，PersonBuilderDirector：

```java
public PersonBuilderDirector {
    private PersonBuilderFactory personBuilderImp;
    
    pulblic PersonBuilderDirector(PersonBuilderFactory personBuilderImp){
        this.personBuilderImp=personBuilderImp;
    }
    
    public Person buildPerson(){
		personBuilderImp.createHead();
		personBuilderImp.createBody();
		personBuilderImp.createLegs();
		return personBuilderImp.buildPerson();
	}

}
```

最终使用方法、测试代码：

```java
public static void main(String[] args) {
		ThinPersonBuilder thinPersonBuilder = new ThinPersonBuilder();
    //PersonBuilderFactory thinPersonBuilder = new ThinPersonBuilder(); 这样写也可以
		PersonBuilderDirector director = new PersonBuilderDirector(thinPersonBuilder);
		Person thinPerson = director.buildPerson();
		System.out.println(thinPerson.toString());
		    
    	//假设来了个新功能，需要制造胖人，则可以继续这么写，
    	//而不用改动现有代码
		FatPersonBuilder fatPersonBuilder = new FatPersonBuilder();
		PersonBuilderDirector director2 = new PersonBuilderDirector(fatPersonBuilder);
		Person fatPerson = director2.buildPerson();
		System.out.println(fatPerson.toString());
	}
```

经过这么一番实践，越发觉得，和抽象工厂模式很像，但又有不同。总之，**抽方法、抽接口、抽抽象类！**。

### 1.5.3 Java源码中的建造者模式



## 1.6 原型模式



# 2 结构型设计模式

## 2.1 适配器模式

## 2.2 装饰器模式

## 2.3 代理模式

## 2.4 外观模式

## 2.5 桥接模式

## 2.6 组合模式

## 2.7 享元模式

# 3 行为型设计模式

## 3.1 策略模式

## 3.2 模板方法模式

## 3.3 观察者模式

## 3.4 迭代子模式

## 3.5 责任链模式

## 3.6 命令模式

## 3.7 备忘录模式

## 3.8 状态模式

## 3.9 访问者模式

## 3.10 中介者模式

## 3.11 解释器模式

