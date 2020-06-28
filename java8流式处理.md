
@[TOC]
# 流式处理简介
java8，即jdk1.8，中新增的内容，一般情况下，对一个集合中的所有元素操作，经常使用循环遍历的方法，比如，现在有Student类如下：
```java
@Getter
@Setter
public class Student {
    private int number;
    private int age;
    private int grade;
    private String name;

    public static Student newRandomStudent(){
        Student student=new Student();
        student.setNumber(CommonUtil.generateSpecificNumbers(6));
        student.setAge(CommonUtil.generateSpecificNumbers(1));
        student.setGrade(CommonUtil.generateSpecificNumbers(0));
        student.setName(CommonUtil.generateSpecificString(5));

        return student;
    }
}

```
该类有几个基础数据类型的成员变量，和一个静态方法用于生成一个随机实例，同时有List<Student> list，为了得到list中所有age为偶数的学学生列表可以使用如下传统的循环遍历方法：
```java
    public List<Student> getAstudentListWhoseAgeIsOushu(List<Student> list) {
        List<Student> result = new ArrayList<>();

        if (list == null) {
            return result;
        }

        for (Student everyStudent : list) {
            if (everyStudent.getAge() % 2 == 0) {
                result.add(everyStudent);
            }
        }

        return result;

    }

```
而在java8中的利用流处理的特性，可以使用如下方法处理：
```java
    public List<Student> getAstudentListWhoseAgeIs_Oushu_2(List<Student> list) {
        List<Student> result = new ArrayList<>();

        result = list.stream()   //集合转换为Stream类，能转换的还有数组、文件
                .filter(entry -> entry.getAge() % 2 == 0)   //中间操作，Stream的实例方法，函数式写法，还有其余多种中间操作
                .collect(Collectors.toList());  //终端操作，对中间操作产生的新Stream的后续处理，本质仍然是Stream的实例方法，
        //只要是Stream的实例方法，都可以直接往后接；可以追加多个Stream方法进行处理

        return result;

    }
```
流处理，本质上就是将一collector、arrat、file实例通过stream()方法转换Stream<T>类型的流实例，再通过实例方法进行一系列处理，实例方法的返回值可以是原本的流，也可以是经过映射改变了泛型类型的流，因此，流处理可以使用“一长条”语句就完成了要写好几行的循环结构，通过前面的流处理方法的示例，可以总结出，流处理一般分为三个步骤：流化（将其他类型的实例转换为泛型的流）、中间操作（过滤、去重、映射等）、输出（得到一个非流的实例或者多个非流实例的集合或者数组）。
因此，从流化、中间操作、输出操作三个部分分别介绍。
# 流化
动态数组List<T>、集合Set<T>等，都直接可以通过实例方法stream()转化为流，主要是通过实现基类Collection的stream()方法：
```java
default Stream<E> stream() {
        return StreamSupport.stream(spliterator(), false);
    }
```
# 中间操作
## filter()
过滤操作， 通过lambda表达式，逐个处理流中的每个条目（entry)，过滤掉不符合的条目，生成的新流是filter()的返回结果，后续可对该新流进行进一步的处理，任然使用Stream<T>的实例方法或者静态方法。
## distinct()
去重操作，调用Stream<T>中T类的equal()方法进行相等判断，自定义类务必要重写equal()方法和hash值方法，确保过滤有效。
## limit()
limit(n)，只取流中的前n个条目，不足n时，按照实际条目数返回流实例。
## skip()
skip(n)，跳过流中的前n个条目，总条目数不足n时，返回空的流实例。
## sort()
直接看用法：
```java
    public List<Student> getAstudentList_age_cong_da_dao_xiao_pai_xu(List<Student> list) {
        List<Student> result = new ArrayList<>();

        result = list.stream()   //集合转换为Stream类，能转换的还有数组、文件
                .sorted((o1, o2) -> o2.getAge() - o1.getAge())   //中间操作，Stream的实例方法，函数式写法，还有其余多种中间操作
                .collect(Collectors.toList());  //终端操作，对中间操作产生的新Stream的后续处理，本质仍然是Stream的实例方法，
        //只要是Stream的实例方法，都可以直接往后接；可以追加多个Stream方法进行处理

        return result;

    }
```
## map()
先看一下用法：
```java
    public List<String> getListMappedByName(List<Student> list){
        List<String> result=new ArrayList<>();

        result=list.stream()
                .map(Student::getName) //将Stream中的每个entry，映射为entry的Name成员属性，每个entry的类型变为Name属性的String类
                .collect(Collectors.toList());

        return result;
    }
```
就是将原本流中一个完成的条目，映射成别的数值、对象实例，本方法将一整个完整的学生映射成这个学生的name，因为name是String类型，因此map的返回值是Stream<String>，之后再机型toList()操作时，返回的list类型也是List<String>。
## flatmap()
将Stream<Object[]>扁平化为Stream<Object>，比如Stream<String[]>扁平化映射为Stream<Stream>，从而更加方便后续的操作，比如后续的distinct（），只能针对String进行去重，而不能针对String[]，因为String[]没有重写equal方法，去重时比较的是引用地址。
# 输出操作
输出操作是流处理的最后一步，可能是一下几种操作。
## 查找
allMatch，是否全部都满足指定的参数行为，返回值为布尔值，如：
```java
boolean allOver18 = students.stream().allMatch(student -> student.getAge() >= 18);
```
anyMatch，是否存在一个或多个满足指定的参数行为，返回值同样为布尔值，如：
```java
boolean someoneOver18= students.stream().anyMatch(student -> student.getAge() >= 18);
```
noneMatch，是否不存在满足指定行为的元素，返回值为布尔值，如：
```java
//是否没一个学生是18岁以上的
boolean noneIsOver18 = students.stream().noneMatch(student -> student.getAge() >= 18);
```
findFirst，必须配合filter(）使用，返回满足条件的第一个元素，返回值是Stream<T>对应的Optiona<T>，如：
```java
Optional<Student> optStu = students.stream().filter(student -> student.getAge() >= 18).findFirst();
```
findAny，findAny相对于findFirst的区别在于，findAny不一定返回第一个，而是返回任意一个，因此，对于顺序流式处理，findFirst和findAny返回的结果是一样的，当并行流式处理的时候，查找第一个元素往往会有很多限制，如果不是特别需求，在并行流式处理中使用findAny的性能要比findFirst好。
## 归约
参考：<https://www.cnblogs.com/shenlanzhizun/p/6027042.html>
```java
//非规约的方法
int totalAge = students.stream()
                .filter(student -> "计算机科学".equals(student.getMajor()))
                .mapToInt(Student::getAge).sum();//mapToInt(Student::getAge)本质就是mapToInt(entry->entry.getAge())
// 归约操作
int totalAge = students.stream()
                .filter(student -> "计算机科学".equals(student.getMajor()))
                .map(Student::getAge)
                .reduce(0, (a, b) -> a + b);

// 进一步简化
int totalAge2 = students.stream()
                .filter(student -> "计算机科学".equals(student.getMajor()))
                .map(Student::getAge)
                .reduce(0, Integer::sum);

// 采用无初始值的重载版本，需要注意返回Optional
Optional<Integer> totalAge = students.stream()
                .filter(student -> "计算机科学".equals(student.getMajor()))
                .map(Student::getAge)
                .reduce(Integer::sum);  // 去掉初始值
```
## 收集
前面利用collect(Collectors.toList())是一个简单的收集操作，是对处理结果的封装，对应的还有toSet、toMap，这些方法均来自于java.util.stream.Collectors，称之为收集器。收集器除了基本的Stream<T>封装功能外，还具有归约、分组、分区功能。
### 归约
收集器也提供了相应的归约操作，但是与reduce在内部实现上是有区别的，收集器更加适用于可变容器上的归约操作，这些收集器广义上均基于Collectors.reducing()实现。
```java
//求学生的总人数
long count = students.stream().collect(Collectors.counting());
// 进一步简化
long count = students.stream().count();
//-------------------------------------------------

// 求最大年龄
Optional<Student> olderStudent = students.stream().collect(Collectors.maxBy((s1, s2) -> s1.getAge() - s2.getAge()));
// 进一步简化
Optional<Student> olderStudent2 = students.stream().collect(Collectors.maxBy(Comparator.comparing(Student::getAge)));
//------------------------------------------------

// 求最小年龄
Optional<Student> olderStudent3 = students.stream().collect(Collectors.minBy(Comparator.comparing(Student::getAge)));
//----------------------------------------------

//求年龄总和
int totalAge4 = students.stream().collect(Collectors.summingInt(Student::getAge));
//对应的还有summingLong、summingDouble。
//------------------------------------------

//求年龄的平均值
double avgAge = students.stream().collect(Collectors.averagingInt(Student::getAge));
//对应的还有averagingLong、averagingDouble。
//----------------------------------------

//一次性得到元素个数、总和、均值、最大值、最小值
IntSummaryStatistics statistics = students.stream().collect(Collectors.summarizingInt(Student::getAge));
//输出：
//IntSummaryStatistics{count=10, sum=220, min=20, average=22.000000, max=24}
//对应的还有summarizingLong、summarizingDouble。
//---------------------------------------

//字符串拼接
String names = students.stream().map(Student::getName).collect(Collectors.joining());
// 输出：孔明伯约玄德云长翼德元直奉孝仲谋鲁肃丁奉
String names = students.stream().map(Student::getName).collect(Collectors.joining(", "));
// 输出：孔明, 伯约, 玄德, 云长, 翼德, 元直, 奉孝, 仲谋, 鲁肃, 丁奉
```
### 分组
在数据库操作中，通过GROUP BY关键字对查询到的数据进行分组，java8的流式处理通过Collectors.groupingBy来操作集合。比如按学校对上面的学生进行分组：
```java
Map<String, List<Student>> groups = students.stream().collect(Collectors.groupingBy(Student::getSchool));
```
groupingBy接收一个分类器Function<? super T, ? extends K> classifier，我们可以自定义分类器来实现需要的分类效果。

上面演示的是一级分组，我们还可以定义多个分类器实现 多级分组，比如我们希望在按学校分组的基础之上再按照专业进行分组，实现如下：
```java
Map<String, Map<String, List<Student>>> groups2 = students.stream().collect(
                Collectors.groupingBy(Student::getSchool,  // 一级分组，按学校
                Collectors.groupingBy(Student::getMajor)));  // 二级分组，按专业
```
实际上在groupingBy的第二个参数不是只能传递groupingBy，还可以传递任意Collector类型，比如我们可以传递一个Collector.counting，用以统计每个组的个数：
```java
Map<String, Long> groups = students.stream().collect(Collectors.groupingBy(Student::getSchool, Collectors.counting()));
```
如果我们不添加第二个参数，则编译器会默认帮我们添加一个Collectors.toList()，比如分组的第一个用法，根据学校进行分类。
### 分区
分区可以看做是分组的一种特殊情况，在分区中key只有两种情况：true或false，目的是将待分区集合按照条件一分为二，java8的流式处理利用ollectors.partitioningBy()方法实现分区，该方法接收一个谓词，例如我们希望将学生分为武大学生和非武大学生，那么可以实现如下：
```java
Map<Boolean, List<Student>> partition = students.stream().collect(Collectors.partitioningBy(student -> "武汉大学".equals(student.getSchool())));
```
分区相对分组的优势在于，我们可以同时得到两类结果，在一些应用场景下可以一步得到我们需要的所有结果，比如将数组分为奇数和偶数。
