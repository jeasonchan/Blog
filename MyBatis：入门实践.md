# 1 前言

参考文章：

Mybatis教程-实战看这一篇就够了    https://blog.csdn.net/hellozpc/article/details/80878563

只看上面的实战还远远不够，还需了解这个数据库中间件是如何与JDBC中定义的接口、数据库的驱动类/各种实现类 进行对接的？默认的连接池是啥，及其原理？


# 2 先从JDBC代码谈起
先回顾一下JDBC的用法：

```java
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.PreparedStatement;
import java.sql.ResultSet;

public class JDBCTest {
    public static void main(String[] args) throws Exception {
        Connection connection = null;
        PreparedStatement prepareStatement = null;
        ResultSet rs = null;

        try {
            // 加载驱动
            Class.forName("com.mysql.jdbc.Driver");

            // 获取连接
            String url = "jdbc:mysql://127.0.0.1:3306/ssmdemo";
            String user = "root";
            String password = "123456";
            //现在比较提倡直接使用DataSource获取连接
            connection = DriverManager.getConnection(url, user, password);


            // 获取statement，preparedStatement
            // preparedStatement又叫JDBC存储过程，SQL会先预编译，
            // 同一个preparedStatement反复执行只要换参数即可
            String sql = "select * from tb_user where id=?";
            prepareStatement = connection.prepareStatement(sql);
            
            // 设置参数
            // 注意！！！索引从1开始！！！
            prepareStatement.setLong(1, 1l);
            
            // 执行查询
            rs = prepareStatement.executeQuery();
            
            // 处理结果集
            while (rs.next()) {
                System.out.println(rs.getString("userName"));
                System.out.println(rs.getString("name"));
                System.out.println(rs.getInt("age"));
                System.out.println(rs.getDate("birthday"));
            }
        } finally {
            // 关闭连接，释放资源
            if (rs != null) {
                rs.close();
            }
            if (prepareStatement != null) {
                prepareStatement.close();
            }
            if (connection != null) {
                connection.close();
            }
        }
    }
}
```

我个人比较喜欢写JDBC，因为所有的过程都掌握在自己的代码里，比较自信。但是JDBC，还是有几个硬性问题的，尤其是在商业、大量数据库交互的环境里:

1. 有人会说，sql的用户名、密码什么的直接写在代码里了，时硬编码……然而，不敢苟同，现在完全可以写在配置文件里，然后通过配置文件实例化得到DataSource实例，从而进一步获取各种连接（单次连接、池连接、分布式连接）和各种事务，因此，**JDBC同样可以做到从配置文件得到数据库连接**。

2. sql和Java代码耦合……同样不敢苟同，同样可以自定义一个XML，XML是个sql的语句合集，给每个语句起个名字，java里调用XML反序列化得到的实例即可。想sql和java代码分离，还是看个人，也是有办法的

3. 使用JDBC存储过程时，填充占位的？时，确实需要类型判断，根据不同的类型调用不同的set方法，同时下表也要自己记住，JDBC存储过程的类型不同，还需要调用不同的executeXXXX()方法……这点确实比较麻烦

4. 对于查询语句，对ResultSet的结果，按例的类型取值时，确实要看表结果，表结果一改，Java代码就要改了


为了改进以上问题，考虑使用mybatis！

# 3 MyBatis介绍
MyBatis 是一款优秀的持久层框架，它支持自定义 SQL、存储过程以及高级映射。MyBatis 免除了几乎所有的 JDBC 代码以及设置参数和获取结果集的工作。MyBatis 可以通过简单的 XML 或注解来配置和映射原始类型、接口和 Java POJO（Plain Old Java Objects，普通老式 Java 对象）为数据库中的记录。

官方文档    https://mybatis.org/mybatis-3/zh/index.html

![mybatis框架结构.png](./resources/mybatis框架结构.png)

从框架图可以看出，MyBatis作为持久层的中间件，很好的解决了JDBC中很多无法简单做到（自己想做到也是可以的，造点轮子）的事情。

先来看一下的MyBatis的入门实践，对MyBatis有个认知，高级的高级用法如事务控制什么的，再慢慢深入。

## 3.1 编写配置文件

连接池配置文件,要注意的写在XML文件里的注释

```xml
<?xml version="1.0" encoding="UTF-8" ?>

<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<!--上面的http表明了当前xml文件的标签约束配置，已经包含进jar中了，因为引入了 org.mybatis jar 包。
可见，java的文件索引就是跟Linux一样的，以文件树的形式分布/索引！每个项目的根目录都是一个文件书的起始位置！-->



<!-- 根标签 -->
<configuration>

    <!--  能共用的值  -->
    <properties>
        <property name="driver" value="com.mysql.cj.jdbc.Driver"/>
        <property name="url"
                  value="jdbc:mysql://localhost:3306/?serverTimezone=GMT%2B8"/>
        <property name="username" value="root"/>
        <property name="password" value="123456"/>


    </properties>

    <!-- 环境，可以配置多个，default：指定采用哪个环境 -->
    <environments default="test">


        <!-- id：唯一标识 -->
        <environment id="test">
            <!-- 事务管理器，JDBC类型的事务管理器 -->
            <transactionManager type="JDBC"/>
            <!-- 数据源，池类型的数据源 一般情况下会有普通连接、池连接、分布式连接三种实现-->
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.cj.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql://localhost:3306/?serverTimezone=GMT%2B8"/>
                <property name="username" value="root"/>
                <property name="password" value="123456"/>
            </dataSource>
        </environment>


        <environment id="development">
            <!-- 事务管理器，JDBC类型的事务管理器 -->
            <transactionManager type="JDBC"/>
            <!-- 数据源，池类型的数据源 -->
            <dataSource type="POOLED">
                <property name="driver" value="${driver}"/> <!-- 配置了properties，所以可以直接引用 -->
                <property name="url" value="${url}"/>
                <property name="username" value="${username}"/>
                <property name="password" value="${password}"/>
            </dataSource>
        </environment>
    </environments>


    <!--  配置进mapper文件，即映射文件  -->
    <mappers>

        <!--  以资源路径为根目录的相对路径  -->
        <mapper resource="mybatis/mybatis-mapper.xml"/>

    </mappers>

</configuration>
```

SQL语句的映射合集：

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<!-- mapper:根标签，namespace：命名空间，随便写，一般保证命名空间唯一 -->
<mapper namespace="MyMapper">
    <!-- statement，内容：sql语句。id：唯一标识，随便写，在同一个命名空间下保持唯一
       resultType：sql语句查询结果集的封装类型,tb_user即为数据库中的表
     -->
    <select id="selectAuthor" resultType="java.lang.String">
        select author from test.table1 where id = #{id}
    </select>


</mapper>
```


注意：

1. 配置文件里的路径都是相对路径，相对的根据经是classPath，即为jar包的根目录

## 3.2 Java代码使用

Main.java

```java
package default_package.mybatis练习;

import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;

import java.io.BufferedInputStream;
import java.io.IOException;
import java.io.InputStream;
import java.nio.charset.StandardCharsets;

public class Main {
    public static void main(String[] args) throws IOException {

        /*
        第一步，读取配置文件
         */
        String myBatisConfig = "mybatis/mybatis-config.xml";

        //正如注解所说，从类加载的地方寻找资源
        InputStream myBatisStream = ClassLoader.getSystemResourceAsStream(myBatisConfig);

        //也可以直接使用mybatis里提供的类读取文件
        // Resources.getResourceAsStream(myBatisConfig);

        //标记一下，为重复读取做一个书签
        ((BufferedInputStream) myBatisStream).mark(0);
        System.out.println("配置文件是：\n" + new String(myBatisStream.readAllBytes(),
                StandardCharsets.UTF_8));
        //读取游标重新设置到书签处，以供后续的重复读取
        ((BufferedInputStream) myBatisStream).reset();


        /*
        第二步，根据配置文件，按建造者模式，产生工厂实例
         */

        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(myBatisStream);


        /*
        第三步，执行mapper中的语句
         */
         // openSession方法与很多重载，有的可以用来打开的事务session
        SqlSession sqlSession = sqlSessionFactory.openSession();
        
        //以map里的命名空间和sql的ID进行调用
        System.out.println((String) sqlSession.selectOne("MyMapper.selectAuthor", 4));

        //操作完要对sqlSession进行close()

        //如果使用了非自动提交的session（也就是事务session），需要手动控制提交、回滚


    }
}

```

##　3.3 步骤小结

1. 配置mybatis-config.xml 全局的配置文件 (1、数据源，2、外部的mapper)

2. 创建SqlSessionFactory

3. 通过SqlSessionFactory创建SqlSession对象

4. 通过SqlSession操作数据库 CRUD

5. 调用session.commit()提交事务

6. 调用session.close()关闭会话


