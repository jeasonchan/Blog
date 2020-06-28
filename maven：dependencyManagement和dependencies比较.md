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
**只有**在dependencies和dependency中声明声明的依赖才会被自动引入！！！包括从父pom中继承而来的dependencies和dependency中声明的依赖，同样会被的自动引入。同样的，就像所有的继承一样，如果子pom和父pom都定义了一样依赖的，版本号和scope肯定是按照子类自己定义来的来的。

# 3 dependencyManagement标签
该标签的内定义的依赖，只是声明了，以后有子pom继承时，子pom的dependencies只需要声明具体的依赖，而不必写明版本号，子pom默认的版本号直接使用父pom的dependencyManagement中定义的版本，如果子pom又明确了自己依赖的版本，则会引入子pom自己定义的版本。
为什么要这样设计呢？
当我们的项目模块很多的时候，我们使用Maven管理项目非常方便，帮助我们管理构建、文档、报告、依赖、scms、发布、分发的方法。可以方便的编译代码、进行依赖管理、管理二进制库等等。
当项目的模块很多时，我们可以抽象父pom来管理子项目的公共的依赖。为了项目的正确运行，必须让所有的子项目使用的依赖项版本一致。
因此，在公共依赖项目POM文件中，就行设置dependencyManagement元素。通过它元素来管理jar包的版本，让子项目中引用一个依赖而不用显示的列出版本号。Maven会沿着父子层次向上走，直到找到一个拥有dependencyManagement元素的项目，然后它就会使用在这个dependencyManagement元素中指定的版本号。
**和dependencies标签重大区别**还是要强调一下：
dependencyManagement里只是声明依赖，并不实现引入，因此子项目需要显示的声明需要用的依赖。如果不在子项目中声明依赖，是不会从父项目中继承dependencyManagement里的依赖；只有在子项目的dependencies中写了该依赖项，并且没有指定具体版本，才会从父项目中继承该项，并且version和scope都读取自父pom;另外如果子项目中指定了版本号，那么会使用子项目中指定的jar版本。



