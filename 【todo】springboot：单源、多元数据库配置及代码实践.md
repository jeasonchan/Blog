# 1 前言

SpringBoot能够快速配置并使用数据库是基于以下流程：

1. 从yml文件序列化得到一些类实例
2. 从类实例中读取数据库相关的配置，从而得到数据库连接池对象或者其更高级的封装对象（比如，JdbcTemplate的实例bean  jdbcTemplate），并放入IOC容器中
3. 通过依赖注入的方式，自动注入相应的实例，供我们调用注入的实例的方法

项目中只需要操作一个数据库时，可以直接配置spring对象的成员对象datasource，也会生成唯一的一个连接池对象，依赖注入时，自动注入这唯一的一个的连接池对象。

当项目中有多个数据库连接时，有多种方式来避免自动注入依赖时，spring不知道该注入哪个实例。

分单数据库和多数据库的情况分别介绍如何配置。

# 2 spring boot配置单数据库

以MySQL8.0和使用原生的JDBC为例，配置的数据库连接池。

首先，pom文件引入依赖：

```xml
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
</dependency>

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-jdbc</artifactId>
</dependency>
```

接着，编写yml文件，配置数据库连接：

```yaml
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/?serverTimezone=GMT%2B8
    username: root
    password: 123456
    driver-class-name: com.mysql.cj.jdbc.Driver
```

jdbc的url我还配置了市区的，要不依然会报错。





