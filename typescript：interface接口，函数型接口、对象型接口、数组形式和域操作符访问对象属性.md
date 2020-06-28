```js
//接口

//对象（类）型的接口
interface Person {
    firstName: string;
    secondName: string;
    repeat: (input: string) => void;//声明方法的输入和返回值
}

//var jeason: Person = {};
//如果不使用断言，类型检查时会报，jeason没有实现Person中定义的属性和方法
var jeason: Person = {} as Person;//使用断言，假装是Person类
jeason.firstName = "jeason";
jeason.secondName = "chan";
jeason.repeat = function (input: string): void {
    console.log(`i am ${this.firstName + " " + this.secondName} and i will ${input}`);
};

jeason.repeat("sleep");

var jackson: Person = {
    firstName: "jackson",
    secondName: "Zhang",
    repeat: function (input: string): void {
        console.log(`i am ${this.firstName + " " + this.secondName} and i will ${input}`);
    }
};
jackson.repeat("eat");



//接口中我们可以将数组的索引值和元素设置为不同类型，
//索引值可以是数字或字符串
interface myMap {
    [index: string]: string  //这里其实时定义了一系列 key value 属性
}
var aMap: myMap = {};
console.log(aMap);  // {}

aMap["a"] = "a_value";
console.log(aMap);  // { a: 'a_value' }

aMap.b = "b_value";
console.log(aMap);  // { a: 'a_value', b: 'b_value' }

//可见！！！！！用这两种索引方式添加属性、索引属性，效果是一样的！！！
//但 aMap.b = "b_value"; 还是更自然、更面向对象一点！！！
//也就是说，接口里的属性，既可以通过数组[]的形式进行赋值、添加（属性），
//也可以，通过域操作符(.)的形式进行赋值、添加（属性）



//函数集合型接口，
//而对象型接口里，同时有属性和方法
interface myFunction_Interdace {
    (arg1: string): string;
    (arg1: string, arg2: number): string;
}

//函数型接口主要用来规定函数的输入、输出
var kkkk = function (hahaha: string): string {
    return "";
}


var ahahha: myFunction_Interdace = (arg1: string) => {
    return arg1;
}
```
