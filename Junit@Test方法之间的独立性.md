先看一段代码：
```java
import org.junit.Test;

public class DealIntegerBeanTest {
    private IntegerBean integerBean=new IntegerBean();
    @Test
    public void printInfoOfIntegerBean() {
        new DealIntegerBean().printInfoOfIntegerBean(integerBean);
    }
    @Test
    public void printInfoOfIntegerBean_statcNumber_2() {
        IntegerBean.setStaticNumber(2);
        integerBean.setNotStaticNumber(2);
        new DealIntegerBean().printInfoOfIntegerBean(integerBean);
    }
    @Test
    public void printInfoOfIntegerBean_3() {
        new DealIntegerBean().printInfoOfIntegerBean(integerBean);
    }
}
```
控制台输出是：
```CMD
StaticNumber:1
NoitStaticNumber:1
StaticNumber:2
NoitStaticNumber:2
StaticNumber:2
NoitStaticNumber:1
```
可见，以上3个@Test注解方法之间都是完全独立的，就像DealIntegerBeanTest这类被复制了3份，每份只含有一个@Test方法，然后个自己执行。因此，**非静态属性每次都是全新的，静态属性还是会全局的！！**
上面写法完全等价于：
```java
public class DealIntegerBeanTest {
    private IntegerBean integerBean;
    @Before
    public void setUp() throws Exception {
        integerBean=new IntegerBean();
    }
    @Test
    public void printInfoOfIntegerBean() {
        new DealIntegerBean().printInfoOfIntegerBean(integerBean);
    }
    @Test
    public void printInfoOfIntegerBean_statcNumber_2() {
        IntegerBean.setStaticNumber(2);
        integerBean.setNotStaticNumber(2);
        new DealIntegerBean().printInfoOfIntegerBean(integerBean);
    }
    @Test
    public void printInfoOfIntegerBean_3() {
        new DealIntegerBean().printInfoOfIntegerBean(integerBean);
    }
}
```
第二种写法，使用@Before显式表明，在@Test执行前会执行setup（），从而对对变量integerBean进行初始化。
为了清楚表明@Test方法之前的独立性（静态变量不是独立的），强烈建议使用第二种方法！！并且测试类的编写习惯也是变量只声明，初始化单独在@Test方法中进行或者@Before方法中进行，@Before方法一般放机械性、重复性的初始化代码。

