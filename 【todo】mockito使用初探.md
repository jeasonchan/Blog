写单元测试时，为了要覆盖到一些条件分支或者一些数据必须在生产环境中才能获得，比如数据库的一些操作，不得不“入侵”这个类，使一些方法的返回值成为我们的期望的返回值，要“入侵”类就要使mock，Mockito是几个常用mock框架中的一种。
# 作用
缺点：据说无法mock静态方法、和方法内部new出来的实例的行为
# 实践
## 模拟期望的结果
背景，想对class B进行测试，但是，class  B注入了class A的单例，因此，测试时要为B注入A的单例！
```java
class Hahaha{
//在某class内部
	@mock //用这种注解，就不需要每次都手动A mock_a = mock(A.class);
	A mock_a; //使用注解的方式mock了一个虚拟类的实例,自动new了一个出来，无法调用真实代码，行为完全由自己控制

	@Injectmocks //同样，自动new了一个，但是可以调用真实代码的方法，比如，使用注入的单例
	B mock_b;  	//其余用@Mock（或@Spy）注解创建的mock将被注入到用该实例中，上面mock出来的class A的单例就被
	
	@Test
	public void testSth(){
		MockitoAnnotations.initMocks(this); //或者整个Hahaha类注解“@RunWith(MockitoJUnitRunner.class)”,new mocks
		when(mock_a.isRight(anyString())).thenReturn(true);//规定这个虚拟单例的行为，为class B的测试打下基础
	}
}
```
要点：@Mock，@InjectMocks，when().thenReturn()

---
# 参考
<https://www.cnblogs.com/Ming8006/p/6297333.html>
