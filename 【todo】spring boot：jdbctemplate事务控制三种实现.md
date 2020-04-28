# 1 前言

SpringBoot系列: JdbcTemplate 事务控制      https://www.cnblogs.com/harrychinese/p/SpringBoot_jdbc_transaction.html



TransactionTemplate实现事务即源码浅析   https://blog.csdn.net/songzehao/article/details/95603555



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

多种类型的数据库中间件，自动注入的事务控制器实例会相互干扰， 所以需要创建一个 configuration 类，手动为不同数据源建立对应的 PlatformTransactionManager bean。以使用spring jdbc为例，多数据源配置事务：

```java
public class TransactionManagerConfig{
    
    @Bean    
    @Autowired    //自动注入 dataSource1
    public PlatformTransactionManager platformTransactionManager1(@Qualiter("dataSource1") DataSource dataSource1) {
        return new DataSourceTransactionManager(dataSource1);
        //之后再通过new TransactionTemplate(txManager1)方式生成TransactionTemplate实例
    }
    
    @Bean 
    @Autowired    //自动注入 dataSource2
    public PlatformTransactionManager platformTransactionManager2(@Qualiter("dataSource2") DataSource dataSource2) {
        return new DataSourceTransactionManager(dataSource2);
    }    
}
```
对于使用springboot的jdbc而言，能直接使用的，其实是TransactionTemplate实例，能直接用该实例设置隔离级别，执行事务语句。


如果使用 @Transactional  注解控制事务，则需要指定对应的事务控制器PlatformTransactionManager Bean实例，比如 @Transactional(value="platformTransactionManager1")


接下来，分情况介绍，基于TransactionTemplate和注解饿事务控制。

## 2.1 基于TransActionTemplate
生成 TransactionTemplate 对象时, **至少**需要指定一个 Spring PlatformTransactionManager 接口的实现类，如果向进行事务控制时，还需要的额外指定一个事务配置类，实例代码如下：
```java
this.dataSource = dataSource;//这个是直接从配置文件反序列化得到的

this.dataSourceTransactionManager = new DataSourceTransactionManager(dataSource);
this.defaultTransactionTemplate = new TransactionTemplate(dataSourceTransactionManager)

TransactionTemplate  transactionTemplateWithIsolatoinLevel=new TransactionTemplate(
                this.dataSourceTransactionManager,
                new TransactionDefinition() {
                    @Override
                    public int getIsolationLevel() {
                        return TransactionDefinition.ISOLATION_DEFAULT;
                    }
                }
        );

```

使用的时候，只需要将DML代码封装成一个callback对象, 然后将callback对象传值给TransactionTemplate.execute()方法, 事务控制由TransactionTemplate.execute()完成。

稍微赏析一下TransactionTemplate.execute() 执行有返回值的callback对象的代码:
```java
public <T> T execute(TransactionCallback<T> action) throws TransactionException {
    TransactionStatus status = this.transactionManager.getTransaction(this);
    
    T result;
    
    try {
        result = action.doInTransaction(status);  
        //这里就是我们要放DML代码的地方
        //执行发生异常的话，会自动触发catch里的状态回滚
    
    }catch (RuntimeException | Error ex) {
        // Transactional code threw application exception -> rollback
        rollbackOnException(status, ex);
        throw ex;//注意！！如果当前的catch生效，则这一行的ex并不会下面的捕捉，而是直接从方法怕抛出
    
    }catch (Throwable ex) {
        // Transactional code threw unexpected exception -> rollback
        rollbackOnException(status, ex);
        throw new UndeclaredThrowableException(ex, "TransactionCallback threw undeclared checked exception");
    }
    this.transactionManager.commit(status);
    return result;         
}
```
从上面代码中可以看到, 回滚数据库操作是**自动触发**的，无论是抛出什么类型的异常，并且抛出的异常该可以从execute中捕捉, 当callback对象的doInTransaction函数抛出异常时。或者在doInTransaction函数中可以控制 一个 TransactionStatus 接口的变量(transactionStatus 变量), 该TransactionStatus 接口为处理事务的代码提供一个简单的控制事务执行和查询事务状态的方法,  调用 transactionStatus.setRollbackOnly() 可以回滚事务. 

TransactionTemplate.execute() 使用回调机制传参, 参数类型是 TransactionCallback<T> 接口, 实参可以是:

1、TransactionCallbackWithoutResult **类实例**, 适合于事务没有返回值, 例如save、update、delete等等.

2、TransactionCallback<T> **虚拟类的实现类的实例**, TransactionCallback<T> 泛型中的类型 T 是 doInTransaction() 函数的返回类型, 一般情况下这个 T 类型并不是很重要的, 直接使用 Object 类型即可. 也可以使用将Select的结果保存到这个返回值上.

## 2.1.1 线程间TransactionTemplate实例隔离使用
IOC中只有一个TransactionTemplate实例，如果多个线程都直接使用这唯一的一个实例，并对其设置隔离等级，显然会相互影响，因此，应该使用Threadlocal将该变量进行封装，避免相互影响。代码见：            

https://github.com/jeasonchan/SpringBootDemo/blob/master/multi-datasource-jdbc-connect/src/main/java/com/jeasonchan/dao/DBService1.java

## 2.2 基于注解




## 2.3 手动获取connection对象



