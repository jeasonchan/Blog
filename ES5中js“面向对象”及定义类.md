@[TOC]
# 1  js的“面向对象”
JavaScript是一种基于对象（object-based），一切皆为对象，一切数字、数组什么的都是Number、String等什么的实例。但是，它并不是一种面向对象（OPP）的语言，因为它的语法中没有class（类）（**注意！ES6中已经新增class关键字！为js带来了面向对象的特性！17年的chrome已经支持ES6了，因此，直接使用class其实更加方便。**参考<https://www.cnblogs.com/zcl997136048/p/9283687.html>）。如果我们要把属性（property）和方法（method）封装成一个对象，目前常用的有以下介绍的几种方法。
# 2 创建类的几种方法
## 2.1 生成实例对象的原始模式
即最基本的只能以“手打”的方式生成一个实体类作为模板，再以手打的方式生成这个“类”的实例……可以说完全没有Java或者c++的类的影子……代码实例如下：
```js
var Person = {
    name: new String(),
    age: new Number(),
    country: new String(),
    doSth: printDate()  //这一步js解释器会以为你想把函数返回值作为变量值赋值给  doSth 变量
    //若换成 doSth: printDate，解释器会以为你想把这个 函数名赋值给变量，即 [Function: printDate] 赋值给变量
};     //这是类模板，想要生成相应的类还是要手的

var PersonA = Person;//或者 var PersonA={};  var PersonA=new Object();
PersonA.age = 12;
PersonA.country = "China";
PersonA.name = "jeason";
console.log(PersonA);
```
这是最简单的封装了，将Person的三个基本属性封装在一个对象中。缺点很明显:
1. 你看不出实例和原型之间有什么联系
2. 如果实例多一点，写起来会非常麻烦
3. 无法通过类实例调用类方法（**错误！完全可以如下方式进行方法调用！**）
```js
var person={
	name:"",
	sleep:function(){
		console.log("sleepping!");
	}
}
person.sleep();  //或者 person.sleep;
```

## 2.2 原始模式的改进模式
```js
//原始模式的改进模式
function newPerson(age, country, name) {  //函数名不能和之前的变量名重合，虽然是弱类型语言……
    return { age: age, country: country, name: name };
}
var personA = newPerson(12, "China", "Jeason");
var personB = newPerson(12, "China", "Jackson");
console.log(personA, personB);
```
缺点：
1. 虽然能简化很多重复性的代码输入，但是两个实例并没有内在联系，无法表明出自同一个原型
2. 无法封装方法

## 2.3 构造函数模式
```js
//构造函数模式，解决了从原型对象生成实例的问题
//所谓“构造函数”，其实就是一个普通函数，但是其内部采用了this变量。
//对构造函数使用一次new运算符，就可以生成一个实例，并且this变量会绑定在实例上。
function CoustructorForPerson(age, country, name) {
    this.age = age;
    this.country = country;
    this.name = name;
}
var pesonJeason = new CoustructorForPerson(12, "China", "Jeason");
console.log(pesonJeason.name);
//使用构造函数创建出来的对象，都自动包含的constructor属性，值为构造函数的函数名
//本质上，原始模式对象的constructor其实是Object
console.log(pesonJeason.constructor);  //打印创造自己的真正的构造函数名字/类名
console.log(pesonJeason instanceof CoustructorForPerson);//是否为这个类的实例 true
console.log(pesonJeason instanceof Object);//是否为这个类的实例 true
//可以看出！！所有对象都默认继承于Object，因此都是Object的实例！！和java、c++一样
```
开始有了class的样子，构造函数名就是类型，也通过new操作符从原型创建对象实例。
但是，也有如下缺点：
```js
//构造函数模式缺点:写死的属性和方法会造成内存浪费，比如下面的result和sleep
function PersonTest(name) {
    this.name = name;
    this.sleep = function () {  //在类中定义类实例
        console.log("匿名函数被执行了！")
    };
    this.result="dead";
    this.doSth2 = printDate();
}
var personTest = new PersonTest("Jeason");
personTest.sleep();//调用实例方法
var personTest2 = new PersonTest("Jeason");
//要解决公共属性或者方法的资源浪费问题，采用下面的模式ProtoType模式
```
## 2.4 ProtoType
```js
//ProtoType模式
//JavaScript规定，每一个构造函数 或者 原型 都有一个prototype属性，指向另外一个对象。
//这个对象的所有属性和方法，都被构造函数的实例继承。
//给原型增加prototype属性的方式对上面的类进行改造
function PersonTest2(name) {
    this.name = name;
}
PersonTest2.prototype.result = "dead";//通过类名添加属性
PersonTest2.prototype.sleep = function () {//通过类名添加实例方法
    console.log("sleep函数被执行了！");
};
//再生成实例，实例的type属性和live()方法，其实都是同一个内存地址，
//指向prototype对象，提高了运行效率。
var personTest3 = new PersonTest2("jeason");
console.log(personTest3.result);
console.log(personTest3.sleep());//先打印函数体中的文字，再打印函数的返回值“undefined”
```

```js
//配合protptype而存在的一些方法
//isPrototypeOf，判断某类的prototype属性是否被某实例继承了
console.log(PersonTest2.prototype.isPrototypeOf(personTest3));//true
//hasOwnProperty()，判断是否为自己独有的本地属性，而不是prototype共享的
console.log(personTest3.hasOwnProperty("result"));//false
console.log(personTest3.hasOwnProperty("name"));//true
//in 运算法，用于是否包含该属性，无论本地属性还是prototype属性
console.log("result" in PersonTest2);//fasle，类本身是不包含prototype属性
console.log("result" in personTest3);//true，实例化的时候才会自动包含prototype属性
console.log("sleep" in personTest3);//true
```
注意点：类本身是不包含prototype属性，new出来的实例才包含prototype属性！
