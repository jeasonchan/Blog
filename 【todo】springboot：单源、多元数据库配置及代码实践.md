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

**首先，pom文件引入依赖：**

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

接着，**编写yml文件，配置数据库连接**：

```yaml
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/?serverTimezone=GMT%2B8
    username: root
    password: 123456
    driver-class-name: com.mysql.cj.jdbc.Driver
```

jdbc的url我还配置了时区的，要不依然会报错。

接着，**找一个地方注入jdbcTemplate类实例**，以供我们并使用：

```java
package com.jeasonchan.dailyexercise.database;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.stereotype.Component;

@Component
public class MyJdbcTemplate {
    public JdbcTemplate jdbcTemplate;

    @Autowired
    public MyJdbcTemplate(jdbcTemplate jdbcTemplate) {
        this.jdbcTemplate = jdbcTemplate;
        System.out.println("jdbcTemplate通过构造函数注入完成！");
    }
}

```

其中，使用了@Autowired注解，在构造函数时，自动使用JdbcTemplate类的唯一bean实例，也就是框架自动生成的。如果，自己又手动往IOC里放了一个JdbcTemplate实例，这一步就必然报错了，@Autowired并不知道要使用哪个bean。

最后，模拟**对连接池的使用**，为方便，直接在主函数中使用了：

```java
package com.jeasonchan.dailyexercise;

import com.jeasonchan.dailyexercise.bean.MySqlInfo;
import com.jeasonchan.dailyexercise.bean.SpringConfiguration;
import com.jeasonchan.dailyexercise.database.MyJdbcTemplate;
import com.jeasonchan.dailyexercise.leetcode.二叉树的右视图.Solution;
import com.jeasonchan.dailyexercise.util.SpringUtil;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class DailyExerciseApplication {

    public static void main(String[] args) {
        SpringApplication.run(DailyExerciseApplication.class, args);


        //这一步从IOC中获取bean
        MyJdbcTemplate myJdbcTemplate = (MyJdbcTemplate) SpringUtil.getBeanByName("myJdbcTemplate");
        System.out.println(myJdbcTemplate);
        
        System.out.println(
                myJdbcTemplate.jdbcTemplate.queryForList("show databases;")
        );


    }

}
```

其中，通过这个jdbcTemplate实例，每**单条**SQL执行都可以看作是read commited隔离级别的事务，自动提交。开启事务需要使用额外的类实例。

# 3 spring boot配置多数据库

此处介绍三种方式配置多数据库：

## 3.1 使用Primary表明主数据库



## 3.2 自定义datasource及JdbcTemplate实例



