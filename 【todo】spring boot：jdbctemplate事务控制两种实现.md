# 1 前言

SpringBoot系列: JdbcTemplate 事务控制      https://www.cnblogs.com/harrychinese/p/SpringBoot_jdbc_transaction.html



DataSourceTransActionManager实现事务即源码浅析   https://blog.csdn.net/songzehao/article/details/95603555



在方法上使用事务注解 https://www.cnblogs.com/harrychinese/p/SpringBoot_jdbc_transaction.html



直接从连接池获取connection对象进行事务操作    https://blog.51cto.com/1202955/1926768



# 2 详解及实践

之前使用 JDBC API 操作, 经常用到的对象有: connection 、 preparedStatement、statement，事务的常规套路一般是：

```java
dbConnection.setAutoCommit(false); //transaction block start

//获取statement，然后DML之类的

dbConnection.commit(); //transaction block end

//还有异常捕获和connection.rollback()
```

虽然通过 jdbcTemplate 可以获取 connection 对象，但是我们不能使用 jdbcTemplate.getDataSource().getConnection().rollback() 命令式方式来控制事务,  因为这样获取到 connection 不并一定是执行SQL的那个connection，不仅是连接池，还有Threadlocal的原因。

在Spring中，官方提供直接（也有一些间接、魔改的代码，比如本文的 2.3 部分）提供下面两个方式控制事务:

1. 命令式事务控制方式
   直接使用 TransactionTemplate 类
   特点: 个人觉得 JdbcTemplate + TransactionTemplate 非常搭配，都是轻量级，都是命令式。另外 TransactionTemplate 因为是写代码形式，事务控制做到更细粒度。
2. 声明式事务控制方式 (@Transactional)
   将DB访问封装到 @Service/@Component 类中, 并将具体访问过程放到一个 public 方法中, 并在方法上加上 @Transactional 注解。
   优点: 代码很简洁, 不仅适用于 JdbcTemplate, 而且适用于 Jpa/MyBatis 等数据库访问技术. 
   缺点: 事务控制粒度较粗, 只能做到函数粒度的事务控制, 无法做到代码块级的事务控制, 另外需要理解其背后是通过 AOP + proxy 方式实现的, 使用有比较多的讲究。 



Spring 事务控制方式基础是 PlatformTransactionManager 接口，它为各种数据访问技术提供了统一的事务支持接口，不同的数据技术都有自己事务管理实现：

* Spring JDBC 技术: DataSourceTransactionManager 
* JPA 技术: JpaTransactionManager 
* Hibernate 技术: HibernateTransactionManager 
* JDO 技术: JdoT ransactionManager 
* 分布式事务: JtaTransactionManager  

Spring Boot 项目中，引入了 spring-boot-starter-jdbc 之后，会自动注入一个  DataSourceTransactionManager （PlatformTransactionManager 在JDBC中的实现类）类型 bean 对象，这个对象有两个名称, 分别为  transactionManager 和 platformTransactionManager。

如果又引入了  spring-boot-starter-data-jpa 依赖，则又会自动注入一个 JpaTransactionManager 类型 bean  对象，这个对象有两个名称，分别为 transactionManager 和 platformTransactionManager。

多种类型的数据库中间件，自动注入的事务控制器实例会相互干扰， 所以需要创建一个 TransactionManager configuration 类，手动为不同数据源建立对应的 PlatformTransactionManager bean。以使用spring jdbc为例，多数据源配置事务：

```java
@EnableTransactionManagement
public class TransactionManagerConfig{
    
    @Bean    
    @Autowired    //自动注入 dataSource1
    public PlatformTransactionManager txManager1(@Qualiter("dataSource1") DataSource dataSource1) {
        return new DataSourceTransactionManager(dataSource1);
        //之后再通过new TransactionTemplate(txManager1)方式生成TransactionTemplate实例
    }
    
    @Bean 
    @Autowired    //自动注入 dataSource2
    public PlatformTransactionManager txManager2(@Qualiter("dataSource2") DataSource dataSource2) {
        return new DataSourceTransactionManager(dataSource2);
    }    
}
```



如果使用 @Transactional  注解控制事务，则需要指定对应的事务控制器Bean实例，比如 @Transactional(value="txManager1") . 



## 2.1 基于TransActionTemplate



## 2.2 基于注解



## 2.3 手动获取connection对象



