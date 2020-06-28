@[TOC]
# 0 背景/打桩
**Student.java**
```java

public class Student {
    private String name;
    private int age;
    private String school;

    public Student() {
        //默认的无参构造函数
    }

    public Student(String name, int age, String school) {
        this.name = name;
        this.age = age;
        this.school = school;
    }

    public String getName() {
        return this.name;
    }

    public String getStudentSchool() {
        return this.school;
    }

//    public void setStudentSchool(String school) {  //为了正常序列化特意加上的
//        this.school = school;
//    }

    public int getAge() {
        return this.age;
    }

    @Override
    public String toString() {
        return "Student{" +
                "name='" + name + '\'' +
                ", age=" + age +
                ", school='" + school + '\'' +
                '}';
    }
}

```
# 1 阿里的JSON
**maven依赖**
```xml

```
# 2 Google的Gson
**maven依赖**
```xml

```
# 3 Jackson的ObjectMapper
**maven依赖：**
```xml
 <dependency>
            <groupId>com.fasterxml.jackson.core</groupId>
            <artifactId>jackson-core</artifactId>
            <version>2.8.0</version>
        </dependency>

        <dependency>
            <groupId>com.fasterxml.jackson.core</groupId>
            <artifactId> jackson-databind</artifactId>
            <version>2.8.0</version>
        </dependency>
```
**main方法**
```java
public static void main(String[] args) throws IOException {
        Student student1 = new Student();
        System.out.println("student1:\n" + student1);
        Student student2 = new Student("jeason", 25, "ZJU");
        System.out.println("student2:\n" + student2);

        /**
         * 使用objectMapper进行序列化和反序列化
         */
        ObjectMapper objectMapper = new ObjectMapper();
        objectMapper.configure(SerializationFeature.INDENT_OUTPUT, true);//序列化设置，为true是标准的json串，自带换行
        objectMapper.configure(SerializationFeature.ORDER_MAP_ENTRIES_BY_KEYS, true);
        String student1_o2m_json = objectMapper.writeValueAsString(student1);

        System.out.println(student1_o2m_json);
        /*
        输出：
        {
          "name" : null,
          "age" : 0,
          "studentSchool" : null
        }
         */

        //System.out.println(objectMapper.readValue(student1_o2m_json, Student.class));
        //单纯的使用上面的方式，会报错，因为:
        //1、json串中的 studentSchool 并不是属性名！无法映射为一个属性值
        //2、同时，该属性于是private的，并且又缺少 setStudentSchool() 的方法，更加不能映射了

        //改进方法：增加 public void setStudentSchool(String school)
        //System.out.println(objectMapper.readValue(student1_o2m_json, Student.class));
        //输出：
        //Student{name='null', age=0, school='null'}


        System.out.println("========无脑序列化和反序列化的jackson用法=======");
        //以上objectmapper使用方式有一定的弊端，转换得到的json串中得到属性名依赖get set方法中的名字
        //推荐以下方法，只使用属性名，不使用get set方法
        ObjectMapper objectMapper_recommand = new ObjectMapper();
        objectMapper_recommand.configure(SerializationFeature.ORDER_MAP_ENTRIES_BY_KEYS, true);
        objectMapper_recommand.configure(SerializationFeature.INDENT_OUTPUT, true);
        objectMapper_recommand.setVisibility(PropertyAccessor.GETTER, JsonAutoDetect.Visibility.NONE);//忽略getter方法1
        objectMapper_recommand.setVisibility(PropertyAccessor.SETTER, JsonAutoDetect.Visibility.NONE);//忽略setter方法
        objectMapper_recommand.setVisibility(PropertyAccessor.FIELD, JsonAutoDetect.Visibility.ANY);//所有可见范围的属性都进行序列化
        String student2_json = objectMapper_recommand.writeValueAsString(student2);
        System.out.println(student2_json);
        System.out.println(objectMapper_recommand.readValue(student2_json, Student.class));
		//序列化和反序列化都能完美执行，但是！！这种方法会被迫对全部的属性都进行序列化，并且忽略ignore的注解
    }

```
小结：
1. 不进行特殊配置的情况下，ObjectMapper进行json转化，本身就是比较依赖getter和setter方法中getXXXX和setXXXX的变量名，只想个别属性就行序列化和反序列化时，并且自己并没有使用默认的get和set方法名称，则务必在getter和setter方法的上进行@JsonProperty("xxxx")的注解，得到合适的json串中的属性名，不注解的话，会把get和set后面的名字作为json串中的名称
2. 最推荐的方法：
方法一，使用默认的get和set方法名称，配合在属性上进行ignore注解，实现序列化和反序列化
方法二，没使用默认的get和set方法名时，在get和set方法上@JsonProperty("xxxx")注解对应的属性名
3. 还有很多十分有用的注解，可参考<https://www.iteye.com/blog/a52071453-2175398>
# 4 注意
```java

```
1. **序列化和反序列化一定要用同一种库！！！！！**否则会有意外的错误，比如成员变量包含实例化的Timestamp实例是，Jackson与其他两家的不兼容，无法从json进行反序列化。
