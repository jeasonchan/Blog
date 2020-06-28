@[TOC]
# 1作用域概念
当在 Spring 中定义一个 bean 时，我们可以显式地声明该 bean 的作用域。例如，为了强制 Spring 在每次需要时都产生一个新的 bean 实例，bean 的作用域的属性（attribute关键字为scope）为 prototype。同理，如果我想让 Spring 在每次需要时都返回同一个bean实例，我们应该声明 bean 的作用域的属性为 singleton，bean作用域声明为singleton的话，传统的单例写法就没啥用了，但是！！！！！**无法避免有人调用public的new方法new一个“单例”出来，无法保证绝对的单例！**。
Spring 框架支持以下五个作用域，分别为singleton、prototype、request、session和global session，后三个只支持ApplicationContex容器的WebApplicationContext实现，前两个是通用的。
| 作用域 | 描述 |
|--|--|
|singleton|在spring IoC容器仅存在一个Bean实例，Bean以单例方式存在，默认值|
|prototype|每次从容器中调用Bean时，都返回一个新的实例，即每次调用getBean()时，相当于执行newXxxBean() |
|request|每次HTTP请求都会创建一个新的Bean，该作用域仅适用于WebApplicationContext环境|
|session|同一个HTTP Session共享一个Bean，不同Session使用不同的Bean，仅适用于WebApplicationContext环境|
|global-session|一般用于Portlet应用环境，该运用域仅适用于WebApplicationContext环境|
# 2singleton 作用域
singleton 是默认的作用域，也就是说，当定义 Bean 时，如果没有指定作用域配置项，则 Bean 的作用域被**默认为 singleton**。
当一个bean的作用域为Singleton，那么Spring IoC容器中只会存在一个共享的bean实例，并且所有对bean的请求，只要id与该bean定义相匹配，则只会返回bean的同一实例。
也就是说，当将一个bean定义设置为singleton作用域的时候，Spring IoC容器只会创建该bean定义的唯一实例。
Singleton是单例类型，就是在**创建起容器时就同时自动创建了一个bean的对象，不管你是否使用，他都存在了**，每次获取到的对象都是同一个对象。注意，Singleton作用域是Spring中的缺省作用域。
# 3prototype 作用域
当一个bean的作用域为Prototype，表示一个bean定义对应多个对象实例。
Prototype作用域的bean会导致在**每次对该bean请求（将其注入到另一个bean中，或者以程序的方式调用容器的getBean()方法）时都会创建一个新的bean实例**。Prototype是原型类型，它在我们**创建容器的时候并没有实例化，而是当我们获取bean的时候才会去创建一个对象，而且我们每次获取到的对象都不是同一个对象**。
根据经验，对有状态的bean应该使用prototype作用域，而对无状态的bean则应该使用singleton作用域。




