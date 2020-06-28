@[TOC]
# 1 脚本运行方式
最基本的方法就是通过浏览器的加载html,然后通过浏览器运行其中的js脚本，比如：
```html
(......)
<script>
//js
<\script>
(......)
```
或者
```html
(......)
<script src="./jsFile.js"><\script>
(......)
```
第二种方法，更像一个脚本语言一样去使用js脚本文件：
1. 安装nodejs，nodejs中封装了一个v8引擎，可以解释、执行js脚本
2. 顺便安装一下npm，并设置源为淘宝的源
3. 在终端中使用"node  FileName.js"或者"nodejs FileName.js"这样的语句让nodejs中的v8引擎执行脚本
更推荐第二种方法，通过第二种方式，js脚本甚至可以一定程度上临时替代一下python和shell的执行脚本功能。
# 2 js语法
```javascript
console.log("这是一个js脚本！");

// js字面量
// 一个字面量就是的一个常量，有数字字面量，可以是整数、小数或者科学计数比如打印如下数字字面量：
console.log(3.14);
console.log(123e5);
console.log(1001);

// 还有字符串字面量，字符串必须使用单引号或者双引号
console.log("hello world！");
console.log('你好，世界！');

//还有表达字面量，就是表达式的值作为一个字面量，比如：
console.log(5 + 6);
console.log(5 * 6);

//数组字面量，且支持混合数组，数组内可以包含各种对象和数据类型，比如：
console.log([40, 100]);
console.log([40, 100, 12.5, 'abc', "345"]);

//对象字面量，感觉更像一个结构体……
var me = { firstName: "chan", lastName: "jeason", age: 20 };
console.log(me);

//函数字面量
function getCurrentTime(inputVar) {
    console.log("input var is :" + inputVar)
    return Date();
}
console.log(getCurrentTime(23333));

//js解释器，有时候会j将几行合并在一起解释，为了避免误解释，确保和Java/C++一样的以分号作为断句的习惯！！！！

//js变量，变量用与存储数据值，js使用关键字 var 来定义变量，使用等号进行赋值
var x, y, z;
x = 5;
y = 6;
z = 7;

//变量可以通过变量名访问其值或者对其值进行赋值修改。变量通常是可变的，字面量是恒定的，字面量相当于存储在C++中的静态存储区内存中。
// 变量是名称，字面量是值

//js操作符
z = (x + y) * 10;
console.log(z);

//js关键字：和java和C++差不多

//js数据类型，数字、字符串、数组（js数组可以放入不同数据类型）、对象实例等

//变量必须以字母、$或者_开头，比如，$event变量
var $event = "233333";
console.log($event);

//未使用值进行初始化的变量都是 undefined 的
var undefinedVar;
console.log(undefinedVar);   //打印的值为undefined

//js的数据类型：
// 字符串（String）、数字(Number)、布尔(Boolean)、数组(Array)、对象(Object)、空（Null）、未定义（Undefined，
// 共计有限的7种。
var xVar;
console.log(xVar);
xVar = 5;
console.log(xVar);
xVar = "hello world!";
console.log(xVar);
xVar = '你好，世界！';
console.log(xVar);
xVar = "你好" + "\n" + "世界";
console.log(xVar);

//字符串实示例
var answer = "It's alright";
var answer = "He is called 'Johnny'";
var answer = 'He is called "Johnny"';

//布尔值示例
var trueVar = true;
var falseVar = false;

//js数组，大致有三种声明并赋值的方式：
//方式一：先创建Array对象，再进行赋值
var conuntries = new Array();
conuntries[1] = "Japan";//故意不赋值附一个元素
console.log(conuntries);//打印为：[ <1 empty item>, 'Japan' ]
conuntries[3] = 123;
console.log(conuntries);//打印为：[ <1 empty item>, 'Japan', <1 empty item>, 123 ]

//方式二：创建时就进行集中初始化
var conuntries = new Array("China", "Japan", "USA", 11111, `HAHAHAH`);
console.log(conuntries);//打印为：[ 'China', 'Japan', 'USA', 11111, 'HAHAHAH' ]

//方式三：
var conuntries = ["China", "Japan", "USA", 22222, `HAHAHAH`];
console.log(conuntries);//打印为：[ 'China', 'Japan', 'USA', 22222, 'HAHAHAH' ]

//对象实例，以花括号分隔，内部以键值对的形式定义成属性值，如：
var person = { contry: "China", age: 20, color: "yellow" };
//也可以以逗号为分割点，转行定义
//对象属性索引有两种方式：
console.log(person.contry);//像C++或者Java一样，以域操作符的方式索引属性值
console.log(person["color"]);//像数组或者map一样取值
try {
    console.log(person[color]);//这里的color会被当做变量，从而报color未定义的异常
} catch (e) {
    console.log(e);
}

//Undefined 这个值表示变量不含有值，可以通过将变量的值设置为 null 来清空变量。
var temp;
console.log(temp);//打印：undefined
temp = null;
console.log(temp);//打印：null

/*undefined和null对比：
1. 相同点
if 判断语句中，两者都会被转换为false

2. 不同点
Number转换的值不同，Number(null)输出为0, Number(undefined)输出为NaN；

null表示一个值被定义了，但是这个值是空值，作为函数的参数，表示函数的参数不是对象；

作为对象原型链的终点 （Object.getPrototypeOf(Object.prototype)），定义一个值为null是合理的，但定义为undefined不合理（var name = null）；　

undefined表示缺少值，即此处应该有值，但是还没有定义，变量被声明了还没有赋值，就为undefined；

调用函数时应该提供的参数还没有提供，该参数就等于undefined；

对象没有赋值的属性，该属性的值就等于undefined；

函数没有返回值，默认返回undefined；

总结！该有的，却没有，即为undefined。
*/

//声明变量类型
var stringObject = new String;//声明了一个字符串对象
console.log(stringObject);//打印：[String: '']，表情时String对象，值为''
stringObject = null;
console.log(stringObject);//打印：null
stringObject = undefined;
console.log(stringObject);//打印：undefined
stringObject = "";
console.log(stringObject);//打印：（实际上打印了一个空字符串）

var numberObject = new Number;
console.log(numberObject.valueOf());//调用Number对象的valueOf()方法，得到该包装类的值
console.log(numberObject);//打印：[Number: 0]，值为0的Number对象

var boolenObject = new Boolean;
console.log(boolenObject);//打印：false

var arrayObject = new Array;
console.log(arrayObject);//打印：[]，空数组，区别于null和undefined

var object = new Object
console.log(object);//打印：{}，空对象

//js里的变量基本都是全局变量！函数中显式声明的变量和java和c++一样，也不是全局变量。非显式声明的，默认为全局变量，如：
function myFunction(){
	myCar="BMW";//默认为全局变量，整个js文件内都是可见可访问的，生命周期和全局变量一样，直到网页关闭
	var yourCar="BYD";//只在方法内部可见生命周期只在方法执行期间
}

//js变量均为对象，当显式地声明为某一类型对象时，就创建个了一个对象，比如：
// var car ="hahahah"; 其实就是一个String包装类实例
// car.valueOf();调用包装类实例的方法
```
