@[TOC]
# 单元测试UT
单元：可独立测试的基本元素，在逻辑上是独立的部分，比如，一个或者多个函数，一个或多个类，jar/dll/ocx的组件，功能模块。

单元测试：
测试单元功能+单元接口和边界条件+单元局部数据结构+单元各类错误处理路径

单元测试模型，单元测试通过构造隔离的测试环境实现测试，这个封闭的环境主要是模拟被测单元的调用和所依赖的方法，即驱动程序和桩程序：
* 驱动程序，负责给被测单元输入数据，并接受被测单元的返回值；
* 桩程序，模拟被测单元的从属模块，模拟被被测单元调用，给被测单元返回值；

TDD：测试驱动开发，现根据需求写好测试程序，再以满足测试程序需求为目标，编写业务逻辑实现代码。

一个正式的编写好的单元测试用例的特点是：已知输入和预期输出，每一项需求都至少需要两个单元测试用例：正验证和负验证。

每个单元测试可以都可以包含自己的Main函数，运行单元测试时使用即可，整个项目的正式入口类指定即可。
## Junit测试框架
特性：
* 测试工具，一整套固定的工具，保证每次测试时初始化的环境（基准环境）都是相同的，因此，必须包括setUp()方法和tearDown()方法。setUp用来初始化资源、数值什么的，tearDown用来释放资源。
```
import junit.framework.TestCase;

public class TestToolsExample extends TestCase { //继承于一个抽象类
    public static void main(String[] arg){
        TestToolsExample testToolsExample=new TestToolsExample();
        try{
            testToolsExample.setUp();
        }catch (Exception e){
            e.printStackTrace();
        }finally {
            testToolsExample.testAdd();//最基础的调用测试方法，不过一般使用TestRunner进行调用
        }
    }

    //测试工具示例
    protected int value1, value2;

    @Override
    protected void setUp() throws Exception {  //初始化测试环境
        super.setUp();
        value1 = 3;
        value2 = 3;
    }

    public void testAdd() { //不是一个main函数，为什么idea可以将其识别为可运行的方法
        int result=value1+value2;
        assertTrue(result==6); //一种断言用法

    }
}
```
* 测试套件，捆绑几个测试案例同时运行，@RunWith和@Suite都被用过运行测试套件，例子如下：
```
import org.junit.runner.RunWith;
import org.junit.runners.Suite;

//Junit Suite Test
@RunWith(Suite.class)
@Suite.SuiteClasses({JunitToTest1.class, JunitToTest2.class})  //注解
public class JunitSuiteTest {
	//Do nothing，这个类的作用主要就是两个测试联合一起测试
}

//第二个java文件
import org.junit.Assert;
import org.junit.Test;

public class JunitToTest1 {//本类的目的是测试MessageUtil中的printMessage方法
    String message = "Robert";
    MessageUtil messageUtil = new MessageUtil(message);

    @Test
    public void test_MessageUtil_printMessage() {
        Assert.assertEquals(message,messageUtil.printMessage());//通过断言判断，实际输出是否和理想值相同
    }
}

//第三个java文件
import org.junit.Assert;
import org.junit.Test;

public class JunitToTest2 {//目的，测试MessageUtil的sayHello方法
    String message="Robert";
    MessageUtil messageUtil=new MessageUtil(message);
    String exceptedMessage="Hi!"+message;
    @Test
    public void test_MeassageUtil_sayHello(){
        Assert.assertEquals(exceptedMessage,messageUtil.sayHello());
    }
}

```
* 测试运行器，用于执行测试案例，好像是直接去执行一个测试类（测试方法的集合），这个测试类可能的包含多个测试方法，可对每个测试方法的测试结果进行输出。核心是runClasses()方法。
* 测试分类，三类重要方法，测试断言（判断实际输出和理想输出是否匹配）、测试用例（运行多重测试工具）、测试结果（收集测试用例的结果并进一步处理）。

### 重要的API
四个重要的API是Assert、TestCase、TestResult、TestSuite。
* Assert，断言方法的集合，单元测试最核心的语句
* TestCase，感觉就是比普通的断言多setUp和tearDown，适用于多次重复测试同一个测试类
```
import junit.framework.TestCase;
import org.junit.Before;
import org.junit.Test;
import org.junit.runner.JUnitCore;
import org.junit.runner.notification.Failure;

public class TestCaseInApiStudy extends TestCase{
    public static void main(String[] arg){//测试启动函数，其实不用这个也行，直接运行@test注解过的方法所在的类
        org.junit.runner.Result result= JUnitCore.runClasses(TestCaseInApiStudy.class);//调用的时候，该类中的所有（转行）
        // 测试方法都会被执行，并且所有的测试得到的结果会按照测试方法的执行顺序写入到result中
        for (Failure failure:result.getFailures()){ //使用runClasses()调用测试方法的套路
            System.out.println(failure.toString());
        }
        System.out.println(result.wasSuccessful());

    }

    protected double value1;
    protected double value2;

    @Before
    @Override
    protected void setUp() throws Exception {
        super.setUp();
        value1=2.0;
        value2=3.0;
    }

    @Override
    protected void tearDown() throws Exception {
        super.tearDown();
    }

    @Test
    public void testAddMethod(){
        System.out.println("Number of TestCase:"+this.countTestCases());//返回当前TestCase被使用的次数
        System.out.println("The name of the TestCase:"+this.getName());  //返回的是当前TestCase的名称
        this.setName("A new name for Test Case");
        System.out.println("The new name of the TestCase:"+this.getName());
    }

    @Test
    public void testAddMethod2(){
        System.out.println("Number of TestCase:"+this.countTestCases());//返回当前TestCase被使用的次数
        System.out.println("The name of the TestCase:"+this.getName());  //返回的是当前TestCase的名称
        this.setName("A new name for Test Case");
        System.out.println("The new name of the TestCase:"+this.getName());
    }

}

```
* TestSuite，联合多个测试类，同时运行多个测试类中的多个测试方法，是测试类的集合，而上面的TestCase只是一个测试类
```
import junit.framework.TestResult;
import junit.framework.TestSuite;

public class TestSuiteInApiStudy extends TestSuite {
    public static void main(String[] arg){
        TestSuite testSuite=new TestSuite(JunitAssert1.class,JunitAssert2.class);
        TestResult testResult=new TestResult();
        testSuite.run(testResult);
		System.out.println("Number of test cases = " + testResult.runCount());//从输出的结果里
                              //统计执行的TestCase数量，也可以直接从TestSuite里读出这个数值
    }

}

```
* TestResult，用来收集所有测试案例的结果，是测试结果的集合，比如：
```
import junit.framework.AssertionFailedError;
import junit.framework.Test;
import junit.framework.TestResult;
import org.junit.Assert;
import org.junit.runner.JUnitCore;
import org.junit.runner.Result;
import org.junit.runner.notification.Failure;

public class TestResultInApiStudy extends TestResult {
    public static void main(String[] arg){
        Result result= JUnitCore.runClasses(TestResultInApiStudy.class);//这个主函数全是套路写法
        for(Failure failure:result.getFailures()){
            System.out.println(failure.toString());
        }
        System.out.println(result.wasSuccessful());
    }

    @Override
    public synchronized void addError(Test test, Throwable e) {
        super.addError(test, e);
    }

    @Override
    public synchronized void addFailure(Test test, AssertionFailedError e) {
        super.addFailure(test, e);
    }

    @org.junit.Test
    public void testAddMethod(){ //这里面写测试方法肯定不规范……
        String message="1234";
        Assert.assertEquals(message,"456");//测试方法里有断言，才会产实际有用的对错
    }

    @Override  //用于标记测试结束
    public synchronized void stop() {
        super.stop();  //测试结束
    }
}

```
其中，TestCase和TestSuite为最长操作的对象。TestCase，字面意思，测试用例，一般一个类对应一个TestCase。TestSuite，测试集合，即一组测试，是把多个相关测试归入一组的快捷方式。TestRunner，测试运行器，执行TestSuite的程序，但是，也可以使用JunitCore.runClasses()和TestSuite.run()等方法。
一个典型的例子为：<https://www.cnblogs.com/xuyatao/p/7156243.html>，从最基本的case，构建到suite，并且suite实例的初始化方式和前面所用的方法略有不同。

### 基本套路
正规的基本套路可参考上面的链接，偷懒的，最直接运行方法测试的有以下几种使用套路：
1. 直接使用@Runwith和@Suite注解，一次性运行多个测试类
2. 继承TestCase，覆写setUp()，然后使用断言，直接运行
3. 直接使用断言运行测试
4. 使用JunitCore.runClasss()方法，运行第二和第三写好的测试类，和第1种感觉很相似，都是启动测试的方法
5. 构建suite实例，添加TestCase，然后使用实例的run()方法或者TestRunners的静态run()方法

### 补充
1. TestCase中，setUp()和tearDown()不是必须重写的，但是很有作用！执行顺序为，setUp——测试方法1——tearDown——setUp——测试方法1——tearDown。为每个方法构造相同的环境。
2. 常用的注释：
* @Test，被注释说明的、依附在 JUnit 的 public void 方法可以作为一个测试案例。
* @Before，在 public void 方法加该注释是因为该方法需要在 @test 方法前运行，比如TestCase中的setUp()方法。
* @After，在 public void 方法加该注释是因为该方法需要在 @test 方法后运行，比如TestCase中的tearDown()方法。
* @BeforeClass，给public void 方法加该注释是因为该方法需要在类中**所有**方法前运行，有点像构造函数。
* @AfterClass，在**所有**方法结束后才执行，这个可以用来进行清理TestCase类中所用到的资源。
* @Ignore，被注解的测试方法或者测试类都不会被执行
3. TestSuite运行方式， 目前已知两种：
```
import junit.framework.TestSuite;
import junit.textui.TestRunner;
import org.junit.runner.JUnitCore;
import org.junit.runner.Result;
import org.junit.runner.RunWith;
import org.junit.runner.notification.Failure;
import org.junit.runners.Suite;

@RunWith(Suite.class)
@Suite.SuiteClasses({JunitAssert1.class,JunitAssert2.class})//可变参数列表
public class TwoWaysToStartTestSuite {
    //这是第一种方法，构建完Suite类后，使用静态方法runClasses启动TestSuite
    public static void main(String[] arg){
        Result result= JUnitCore.runClasses(TwoWaysToStartTestSuite.class);
        for(Failure failure:result.getFailures()){
            System.out.println(failure.toString());
        }
        System.out.println(result.wasSuccessful());
    }

    //第二种方法
    public static void anyName(){
        TestSuite testSuite=new TestSuite();
        testSuite.setName("23333");
        testSuite.addTestSuite(JunitTestCase.class);//注意！这种方式必须添加TestCase类
        testSuite.addTest(new JunitTestCase());//加完类型之后，还要注入实例
        TestRunner.run(testSuite);//返回值是TestResult类和之前的runClasses()返回值类型不同
    }
}
```
最通用的还是runClasses()方法，通吃testcase和testsuite，就是不知道Result和GTestResult的后续处理差异大小。
* 时间测试，即测试方法超时，@test(timeout=1000)  ，经过1000毫秒该注释的测试方法还没测试完则表示测试失败，比如，被测方法内部有个死循环。
* 异常测试，测试被测方法是否抛出了想要得到的异常，测试方法的注解实例如：@Test(expected = ArithmeticException.class)
* 参数化测试，Junit4引入的新功能，允许用不同的值对被测单元进行反复测试。

```
//第一个java文件
import lombok.Getter;
import lombok.Setter;

public class CheckIsEvenNumber {
    @Getter
    @Setter
    private int number;

    public boolean isEvenNumber(int number){
        if(number%2==0){
            return true;
        }
        return false;//不能整除则为奇数
    }
}

//第二个java文件
import org.junit.Assert;
import org.junit.Before;
import org.junit.Test;

import org.junit.runner.JUnitCore;
import org.junit.runner.Result;
import org.junit.runner.RunWith;
import org.junit.runner.notification.Failure;
import org.junit.runners.Parameterized;


import java.util.Arrays;
import java.util.Collection;

import static org.junit.Assert.*;

@RunWith(Parameterized.class)//表明是参数化测试类
public class CheckIsEvenNumberTest {
    //测试启动方法
    public static void main(String[] arg) {
        Result result = JUnitCore.runClasses(CheckIsEvenNumberTest.class);
        for (Failure failure : result.getFailures()) {
            System.out.println(failure.toString());
        }
        System.out.println(result.wasSuccessful());
    }

    private int inputNumber;
    private boolean expectedResult;
    private CheckIsEvenNumber checkIsEvenNumber;//被测试的类，这里注入为一个私有成员变量

    public CheckIsEvenNumberTest(int inputNumber, boolean expectedResult) {
        //该测试类的构造函数，因为参数化列表传参构造当前这个测试类时，需要这个构造函数来初始化各个私有成员变量
        this.inputNumber = inputNumber;
        this.expectedResult = expectedResult;
    }

    @Before
    public void initialize() {
        this.checkIsEvenNumber = new CheckIsEvenNumber();
        //这一步确保了下面使用其方法时，指针不为空；
        // 或者 私有成员变量直接new一个也可以，或者构造函数new一个
        //但是，用before初始化更节省资源

    }

    @Parameterized.Parameters //标记这个方法为产生 输入参数和期望输出  的静态方法
    public static Collection AnyName() { //用这个产生测试要用的输入值和期望的结果
        return Arrays.asList(new Object[][]{
                {1, true}, {2, true}, {3, false}, {5, false}, {9, true}
        });//最后一个是期望值，前面的都是输入值
    }

    @Test(timeout = 1000)
    public void realTestMethod() {
        System.out.println("当前使用的参数列表中的数字是：" + this.inputNumber);
        Assert.assertEquals(this.expectedResult,
                this.checkIsEvenNumber.isEvenNumber(this.inputNumber));
    }/*解析：
    测试方法的写法并没有什么太大不同，
    根据this标记可以看出，定义的三个私有变量，是十分有必要的，一个是输入参数，一个是输出结果，一个是被测试的类；

    为了确保输入参数能被初始化到私有成员变量中，还必须构建以输入参数为变量的构造函数

    同时，产生测试要用的数据，还必须有被标记过的、返回值是集合的静态方法

    总结一下：参数化测试，本质上就是同时实例化多个测试类，然后用静态方法中产生的输入参数去初始化各个测试实例，
    所有测试实例准备好之后，再开始运行测试，再用断言对比，再将各个实例的结果和对应的结果对比，
    得出测试结论。知道这个思路之后，甚至可以用手工的方法，自己编写程序直接实现参数化测试
    */


}

```
* TestCase这个类和普通的类的区别：继承TestCase的子类自带setUp和tearDown，自己覆写就会在测试方法前后自动运行，感觉完全等价于自己写个setUp和tearDown方法，然后注解@Before和@After。
* TestCase这个类和普通的类的区别：TestCase中的测试方法，不仅要用@Test注解，还要以“test”作为测试方面名的开始，普通的测试类则没有此限制，用@Test进行注解即可。
* 无显式返回值的单元测试写法<https://blog.csdn.net/u013803262/article/details/57075052>，使用反射对相关Field中的值进行监控，从而判断一些private的方法或者无返回值方法是否有效。
* 

---
# Mockito
制造假对象，假实例，来配合进行单元测试，此mock无法mock静态方法，和powermock和jmock相比，功能较为简单
# PowerMock 
此mock无法mock静态方法
DB数据库相关
oracle
# JMock

