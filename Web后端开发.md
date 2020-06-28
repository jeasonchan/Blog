@[TOC]
# Websevice
Web服务提供一种跨平台的远程调用服务。

服务端和客户端的数据传递. 对象传递遵循soap协议，soap传输协议是基于xml和http的。为了描述我们开发出来的web服务 ，有了wsdl，即为使用标记语言写出的。实现这种协议的方式有jax-ws和jax-rs框架，ws是以动作为驱动的，rs是以名词（资源）位驱动的。基于这两种实现框架，又产生了其他框架，比如同时基于两者的apache的CXF框架。

 ---

# 微服务后端
## 杂七杂八

msb：微服务间通信不是直接点对点发送的，而是先发送至路由

elasticsearch：日志记录用的

rootpath：微服务前缀，/api/微服务名字/版本/  ，通用写法

```java
@Entity  //hibernate将这个类识别为实体类，并可以为其绑定一个数据库对象
@Table(name = "hero")  //hibernate将这个类和以此名字的数据库绑定
@Setter
@Getter
public class Hero {

 @Id  //数据库主键
 @GeneratedValue  //表明是自动生成
 @JsonProperty  //表明能够转化为json串，传入/传出json串时，id和{"id"："24"}对应，24为其值
 private Long id;
 
 @Column(nullable = false)
 @JsonProperty
 private String name;
 
 @Column
 @JsonProperty
 private String age;
 
 @Column
 @JsonProperty
 private String ability;
}
```
restful风格中的get、post、put、delete等操作都是针对url资源的操作，http向服务器发送请求时，会表明自己是什么类型的请求，同时，有QueryParam("name")的，在url中以“?name=hahahah”的形式想资源方法中赋值。
* @get 用户想从服务器获取资源，一些内容显示在url中
* @put
* @post 用户想把东西发给服务器，发送的内容的写在body中，url中是看不到QueryParam的参数
* @delete 用户想删除服务器里的某些资源

```java
@Path("/")  //表示该类提供的restful接口访问的根路径url，真正的url全路径还要加上server里的rootpath。
@Service  //表示该类是个容器类，容器启动时会自动实例化该类。
@Api(tags = {"hero api"})  //api名字会显示为hero api
@Produces(MediaType.APPLICATION_JSON)  //表示restful接口返回的消息格式，此处返回JSON格式的内容。
public class HeroResource {
 private HeroDao heroDao;
}
```
---
体会：
* 基本类，类成员变量被@JsonProperty注解的，可以放在bean package中，每个基本类其实都应该是POJO类
* 基本类的操作类，比如hibernate类的数据库DAO操作类，
* 微服务中的资源类，通过操作类的实例，对POJO类的实例进行操作
使用的时候，先对操作类注入/set一个POJO实例，再将得到的操作实例注入/set到资源类，好处是，耦合度较低。
---
http请求的内容，发送的信息包括url、header（POST、GET、DELETE、PUT）、body（json串）、token，其中，URL就是ip+path+（一些参数）。
response中的{"Code":"200","Message":"hahaha"}，就是位于body中。

---
Reuslt类可以有很多个，不一定要一个类就能完成所有情况的结果展示或者回复，但是，多种不同类要继承于一个父类，方便向上转型。







