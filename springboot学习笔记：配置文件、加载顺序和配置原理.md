# 1 配置文件
使用spring initializr创建spring boot工程时，默认的app全局配置文件是：src/main/resources/application.properties(默认以)，但是，还支持src/main/resources/application.yaml，配置的**名字是固定的**。(很久之前，大多使用XML文件作为配置文件，现在仍然有这中方式。)目前，主流使用yaml书写配置文件。

配置文件的作用是：修改springboot的默认配置（auto configuration）
## 1.1 YAML语法
* key:[**空格**]value

* 层级关系划分：左端对齐的为同一级，上下级之前可以用一个或者多个空格进行缩进划分

* 大小写敏感

* value的类型：

字面量（数字、字符串、布尔）：	字面量所见即所得，直接写即可。但是，双引号的和单引号包括字符串时有稍有区别。

​	英文双引号：不会进行任何转译，真正的所见即所得

​	引文单引号：会像java中的字符串输出，进行转译



对象（属性和值）、Map（键值对）：对象直接写成key:[空格]value的形式，比如：

```yaml
person:
	name: jeason
	age: 16
	country: China
```

或者

```yaml
person: {name: jeason, age: 16, country: China}
```



数组（List、Set）：

使用一个短横表示的数组的中的一个元素的，短横和value之间也必须有空格，比如：


```yaml
animalList:
    - cat
    - dog
    - monkey
```

或者直接使用数组的行内写法：

```yaml
animalList: [cat, dog, monkey]
```



将以上的类型进行综合利用：

```java
@Getter
@Setter
public class Person {
    private String lastName;
    private Integer age;
    private Boolean isBoss;
    private Date birthday;

    private Map<String, Object> maps;
    private List<Object> objectList;
    private Dog petDog;


    @Override
    public String toString() {
        return "Person{" +
                "lastName='" + lastName + '\'' +
                ", age=" + age +
                ", isBoss=" + isBoss +
                ", birthday=" + birthday +
                ", maps=" + maps +
                ", objectList=" + objectList +
                ", petDog=" + petDog +
                '}';
    }
}

```

尤其注意：

Person只配置@ConfigurationProperties注解是无法反序列化并被容器内的实例获取的，必须配合@Component或者直接只使用@EnableConfigurationProperties 。

如果一个配置类只配置@ConfigurationProperties注解，而没有使用@Component，那么在IOC容器中是获取不到properties 配置文件转化的bean。说白了 @EnableConfigurationProperties 相当于把使用  @ConfigurationProperties 的类进行了一次注入。
 测试发现 @ConfigurationProperties 与 @EnableConfigurationProperties 关系特别大。



对应的yaml可写为：

```yaml
person:
  lastName: jeason
  age: 16
  isBoss: true
  birthday: 2017/12/12
  objectList:
    - 123
    - hahahaha
  maps:
    key1: value1
    key2: value2
  petDog:
    name: sb
    age: 666
```

并且，使用如下的依赖，在已经使用prefix="beanName"进行配置bean绑定的情况下，在yaml中编写beanName时，会有提示：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-configuration-processor</artifactId>
    <optional>true</optional>
</dependency>
```



## 1.2 配置文件和类的绑定

最终目的很简单：

1. 以某个类为反序列化模板，可以直接在模板类上使用@configurationproperities，也可以属性中一个一个的使用@value注解，从yaml中取值（$(xxxx)）或者算值（#(xxxx)）

   |                                   | @configurationproperities | @value       |
   | --------------------------------- | ------------------------- | ------------ |
   | 功能                              | 批量注入配置文件中的属性  | 一个一个指定 |
   | 松散绑定（属性名和配置名）        | 支持                      | 不支持       |
   | SpEL                              | 不支持                    | 支持         |
   | JSR303数据校验*                   | 支持                      | 不支持       |
   | 反序列化复杂类型/封装（比如 Map） | 支持                      | 不支持       |

   JSR303数据校验：在模板类上加上@Validated注解，表示要对类的属性进行校验，比如，某属性使用了@Email注解，则在反序列化时，校验字符串是否为Email的格式

   

   **应用场景区别**：

   只是想在业务逻辑中读一下配置文件中的值，可以直接使用@value取值；单纯的编写配置类和配置文件，更推荐使用批量注入的@Configurationproperities。

   

2. 从配置文件中反序列化出一个实例，应该是使用Jackson之类的库，不需要注解

3. 并注入到spring容器中以供使用，在模板类上使用＠component注解即可

## 1.3 加载指定的配置文件@PropertySource

**使用@ConfigurationProperties 注解是，是默认直接读取 resource 文件夹下面的application.yaml（这就是全局配置文件）**。但是！有时候，数据库、redis也有配置文件的需求，都写在一个文件，可能的会写很长，并且，对应的配置类的属性也越来越多。我们就会有多个自定义配置类的需求，并且，就可以在自定义的配置类上**再额外使用**@PropertySource以指定配置文件。

配置类

```java
@Getter
@Setter
@Component  //将该注入到spring容器中，使其能够被获取
@ConfigurationProperties(prefix = "person") //将配置文件中的 person 反序列化成这个类
@PropertySource(value = {"classpath:person.yaml"})
public class Person {
    private String lastName;
    private Integer age;
    private Boolean isBoss;
    private Date birthday;

    private Map<String, Object> maps;
    private List<Object> objectList;
    private Dog petDog;
    }
}
```

对应的配置文件src/main/resources/person.yaml：

```yam
person:
  lastName: jeason_chan
  age: 16
  isBoss: true
  birthday: 2017/12/12
  objectList:
    - 123
    - hahahaha
  maps:
    key1: value1
    key2: value2
  petDog:
    name: sb
    age: 666
```

## 1.4 加载spring的配置文件@ImportResource

在springboot工程中，如果在的资源文件夹里spring的xxxx.xml文件，以希望springboot自动读取这个xml文件，并生成相应的bean，那必定是无法自动实现的………为了让spring应用可以完全迁移到的springboot中，springboot支持的在springboot的应用启动类上使用@ImportResource，以使spring的xml的配置文件生效，比如：

```java
@ImportResource(value = {"classpath:spring.xml"})
@SpringBootApplication //运行该类的main方法来运行应用
public class MainApplication {
    public static void main(String[] args) {
        SpringApplication.run(MainApplication.class, args);
    }
}
```

### 1.4.1向IOC容器中添加组件的方式

spring向容器中添加组件（Bean）的方式（也就是装配方式）有多种：xml文件配置、java配置类、注解配置。到了springboot中，官方推荐使用Java配置类的方式像容器中添加组件，比如：

```java
@Configuration//指定当前类是配置类，来替代之前spring的配置文件；此处，也就是使用Java类作为装配依据
public class MyAppConf {

    @Bean//将方法的返回值放入容器，默认ID是方法名
    public Person person2() {
        return null;
    }
}
```

## 1.5 配置文件的占位符(${...}表达式)

```yaml
person:
  lastName: jeason
  age: 16
  isBoss: true
  birthday: 2017/12/12
  objectList:
    - 123
    - hahahaha
  maps:
    key1: value1
    key2: value2
  petDog:
    name: sb
    age: 666

admin:
  lasNname: ${person.lastName}
  fullNAme: ${person.lastName}chan
  age: ${random.int}
  address: ${person.address:default}
```

总结：

从admin的配置看出来，也就是使用${xxxxx}表达式，取不到值时，可以使用冒号来指定默认值。

## 1.6 Profile

Profile是Spring对不同环境提供不同配置功能的，可以通过激活、指定参数等方式快速切换环境。利用Profile实现多环境切换，针对applicaiton.properties和application.yaml格式，主要有以下2种实现方式。

### 1.6.1 多Profile文件

如果applicaiton.properties，直接通过不同名称的全局配置文件来区别不同的环境，比如：application.properties（默认的配置文件名称必须跟这个一摸一样）、application-a.properties、application-b.properties、application-c.properties。

### 1.6.2 yaml本身支持多文档块方式

使用yaml文档同页多文档特定和profile标注，比如：

```yaml
person:
  lastName: jeason

spring:
  profiles: a  
---
person:
  lastName: jeason
  age: 17

spring:
  profiles: b

```

标注出了不同的环境a和b，a和b的名称是可以自定义的，比如，dev、test、prod什么的。

### 1.6.3 如何激活指定Profile

激活特定的profile的方式也有多种：

1. 直接说的yaml文件的，开头加一个文档，并指明当前激活的profile即可。

```yaml
spring:
  profiles:
    active: a

---
person:
  lastName: jeason
  age: 16

spring:
  profiles: a

---
person:
  lastName: jeason
  age: 16

spring:
  profiles: b

```

2. java参数： --spring.profiles.active=a，既可以作为java -jar xxxxx 的参数，也可以作为idea里面的启动参数。该方式的优先级高于的文档里写的激活。
3. 虚拟机参数：虚拟参数   -Dspring.profiles.active=a

# 2 加载顺序

springboot会自动的扫描四个位置的配置文件，优先级从高到低是：

1. 项目目录/config/application.yaml
2. 项目目录/application.yaml
3. classpath:/config/application.yaml，也就是resource文件夹
4. classpath:/application.yaml，也就是resource文件夹

但是！有冲突时，采用优先级最高的文件里的描述，没有冲突的（尽管优先级最低），还是会和并进最终的配置，这是**互补配置**。



同时还可以通过java参数的形式，使用jar包之外的配置文件，比如：

```sh
$:ls
a.jar     application.yml
$:java -jar a.jar --spring.config.location ./application.yml
```

最终和jar包内的配置文件形成互补配置，在运维时十分常用。



## 2.1 使用外部配置

能够打包进的jar包的文件只有的各种类和classpath下的文件，也就是，**能打包进jar包的配置文件只有，上面提到的3和4**，上面提到的1和2也只有在IDE里点运行时才会有高的优先级。但是！如果要改一次配置文件就必须编译一次的jar包，太麻烦了，因此，有时候还必须使用外部文件、字符串等作为配置输入，和jar内的配置文件形成**互补配置**。

以下开始介绍几种主流的，使用来源于外部的配置信息的方法，配置由高到低的优先级是：

1. 命令行

比如：

```sh
java -jar a.jar --spring.server.port=6666
java -jar a.jar --spring.profiles.active=a
java -jar a.jar --spring.config.location ./application.yml
```

2. jar包**外**的application**-{profile}**.yaml

   jar包和application**-{profile}**.yaml在**同一个目录时**，直接使用javar -jar 运行jar包时，默认**直接使用**在同一个目录中的pplication**-{profile}**.yaml，**没有必要**使用 --spring.config.location ./application.yml 指定配置文件。

3. jar包**内**的application**-{profile}**.yaml

   同上。

4. jar包**外**的application.yaml

   同上。

5. jar包**内**的application.yaml

   同上。

上面的四条就是，**由外向内，profile优先**。

# 3 配置原理