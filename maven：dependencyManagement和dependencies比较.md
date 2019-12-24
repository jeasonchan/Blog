# 1背景
参考<https://www.cnblogs.com/feibazhf/p/7886617.html>。

在看公司项目时，经常在maven的pom配置文件中看到类似如下的结构：
```xml
<parent>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-parent</artifactId>
  <version>2.1.7.RELEASE</version>
<parent/>

<dependencies>
  <dependency>
    <groupId>groupId</groupId>
    <artifactId>artifactId</artifactId>
    <version>1.0.0.RELEASE</version>
  <dependency/>
<dependencies/>

<dependencyManagement>  
          
        <dependencies>  
            <dependency>  
                <groupId>org.eclipse.persistence</groupId>  
                <artifactId>org.eclipse.persistence.jpa</artifactId>  
                <version>${org.eclipse.persistence.jpa.version}</version>  
                <scope>provided</scope>  
            </dependency>  
              
            <dependency>  
                <groupId>javax</groupId>  
                <artifactId>javaee-api</artifactId>  
                <version>${javaee-api.version}</version>  
            </dependency>  
        </dependencies>  
    </dependencyManagement>

```
就像标题中说明的，既有依赖标签，又有依赖管理标签，这两种的所用是什么的呢？

# 2 dependencies标签
**只有**在dependencies和dependency中声明声明的依赖才会被自动引入！！！包括从父pom中继承而来的dependencies和dependency中声明的依赖。同样的，就像所有的继承一
