# 1 @Autowired

使用@Autowired注解警告Field injection is not recommended       https://blog.csdn.net/zhangjingao/article/details/81094529



# 2 @Primary



# 3 @ConfigurationProperties

在@Configuration内部，给Bean方法注解，表示返回的是prefix反序列化得到的实例

```java
@Configuration
public class DBConfig {

    @Bean
    @ConfigurationProperties(prefix = "datasource1")
    public DataSource dataSource1() {
        return DataSourceBuilder.create().build();
    }

}
```





# 4 @Component



# 5 @Bean



# 6 @Configuration
https://blog.csdn.net/icarus_wang/article/details/51649635

使用该注解配置时，配置类不需要再使用@Component










# 7 @Qualifier

百度百科  https://baike.baidu.com/item/%40Qualifier

第三房博客小结  https://www.cnblogs.com/smileLuckBoy/p/5801678.html



@Autowired是按照Class进行搜索，如果该Class只有一个实例，完美，直接注入即可；如果有多个实例Bean，就直接报错。为了解决IOC中有多个实例Bean的情况，使用@Qualifier，切换到按照Bean名字进行注入。



# 8 @Service@Repository@Component等对比

https://www.cnblogs.com/lonecloud/p/5745885.html