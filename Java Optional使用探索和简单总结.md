@[TOC]
# 1 背景
最近在做一个项目，为了方便更新数据库内容，将更新数据库的放抽象为一个公共方法，但是，更新的字段以后有可能会变多，所以想了比较多的方法，比如：必要的参数直接作为固定的几个参数，其余的可弹缩的参数可作为Map类型的输入参数。本来想用 String ... inputs 这样的便长度的参数的，但是这样就没有顺序了……然后想到的optional，不管有没有用，先看看再说。
# 2 基本用法代码实践
optional的出现主要是减少这种垃圾代码出现的频率：
```java
if(obj==null){
	//错误处理机制
}
//正常流程
```
google的guava和java中都自带optional类，功能类似，毕竟openjdk是open的……java中的还融合了null和非null的后处理，还是不错的，看一下optional的代码实践。
```java
public static void main(String[] args) {

        try {

            Integer integerNull = null;
            //直接报空指针异常
//            Optional<Integer> optionalInteger = Optional.of(integerNull);


            Integer integer2 = 1;
            Optional<Integer> optionalInteger2 = Optional.ofNullable(integer2);
            Optional<Integer> optionalIntegerNull = Optional.ofNullable(integerNull);
            System.out.println(optionalIntegerNull.isPresent());//因为optional包住的就是一是一个空指针
            System.out.println(optionalInteger2.get());//如果对保包含null的使用则会抛出空指针异常
            System.out.println(optionalIntegerNull.orElse(666));//如果是null则返回一个默认值

            //通过这样的方式可以直接优雅的抛出异常
            //System.out.println(optionalIntegerNull.orElseThrow());


            //值存在时则使用函数式编程的函数实例
            Consumer<Integer> doSomething = (x) -> {
                System.out.println("原始的值为:" + x + "\n经过自身的次幂为：");
                System.out.println(Math.pow(x, x));
            };

            //本质上就是传上面的doSomething进去
            optionalInteger2.ifPresent((x) -> {
                System.out.println("原始的值为:" + x + "\n经过自身的次幂为：");
                System.out.println(Math.pow(x, x));
            });


            //存在的就话就会执行 doSomething，不存在就会执行后面的runnable实例，
            //也可以直接内联到一起
            Runnable runnable = () -> {
                for (int i = 0; i < 5; i++) {
                    System.out.println(i + ":value is null");
                }
            };

            optionalIntegerNull.ifPresentOrElse(doSomething, runnable);


        } catch (Exception e) {
            System.out.println(e.getMessage());
        }
    }
```
optional和java中的函数式编程结合的很不错，lambda表达其实可以直接内联，但是，为了方便别人接手代码，还是分离出来好。
# 3 Optional的使用建议
参考了<https://www.cnblogs.com/kingsonfu/p/11009574.html>、<https://blog.csdn.net/lifen0908/article/details/89478904>。
尽量避免使用的地方：
1. 避免使用Optional.isPresent()来检查实例是否存在，因为这种方式和null != obj没有区别，这样用就没什么意义了。
2. 避免使用Optional.get()方式来获取实例对象，因为使用前需要使用Optional.isPresent()来检查实例是否存在，否则会出现NPE问题。可以用带“or”的方法。
3. 避免使用Optional作为类、实例的属性，也不要作为方法的参数，而应该在返回值中用来包装返回实例对象。
因为，optional不能进行序列化，json传输不方便，对象就不是POJO了。将该类型作为方法或者构造函数的参数，这将导致不必要的代码复杂化，入参时还要再构造一个对象出来，这点不能太绝对，看个人对方法入参和入参检查的设计能力。使用Optional作为返回值可以增强stream处理，构建流式API. 比如， findFirst()就是返回一个Optional对象。例如：
```java
@Test
public void whenEmptyStream_thenReturnDefaultOptional() {
    List<User> users = new ArrayList<>();
    User user = users.stream().findFirst().orElse(new User("default", "1234"));
    
    assertEquals(user.getEmail(), "default");
}
```
findFirst()方法的返回值是Optional类型，使用了orElse()方法代替了空指针的输出情况。方便！！
4. Optional和steam组合更有益处，级联调用是危险的，很容易产生空指针。比如：
```java
String isocode = user.getAddress().getCountry().getIsocode().toUpperCase();
```
在传统做法里，我们需要不停的进行空指针判断，产生垃圾代码：
```java
    if (user != null) {
        Address address = user.getAddress();
        if (address != null) {
            Country country = address.getCountry();
            if (country != null) {
                String isocode = country.getIsocode();
                if (isocode != null) {
                    isocode = isocode.toUpperCase();
                }
            }
        }
    }
```
使用Optional可以精简代码，降低复杂度：
```java
    String result = Optional.ofNullable(user)
      .flatMap(User::getAddress)
      .flatMap(Address::getCountry)
      .map(Country::getIsocode)
      .orElse("default");
```
# 4 补充
optional和stream都有map和flatmap方法，那map和flatmap的区别：
对于stream，两者的输入都是stream的每一个元素，map的输出对应一个元素，必然是一个元素（null也是要返回），flatmap是0或者多个元素（为null的时候其实就是0个元素）。
flatmap的意义在于，一般的java方法都是返回一个结果，但是对于结果数量不确定的时候，用map这种java方法的方式，是不太灵活的，所以引入了flatmap。

对于Optional的map和flatmap，map是把结果自动封装成一个Optional，但是flatmap需要你自己去封装。 
