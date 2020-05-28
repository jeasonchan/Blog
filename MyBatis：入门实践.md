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

3. 使用JDBC存储过程时，填充占位的？时