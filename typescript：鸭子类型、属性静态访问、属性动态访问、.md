```js
//class 类
class MyClass {
    //属性
    [index: string]: any;

    //构造函数
    //和java一样，默认自带无参数的构造函数
    constructor() {
    }


    //一些方法
}

var myClass_1: MyClass = new MyClass();
myClass_1.keys = "123";//以静态方式添加属性
myClass_1["key2"] = "123";//以动态方式添加属性
myClass_1.sayHi = function (): string { return "hahaha" };
console.log(myClass_1)
//由于该方法是动态添加的，编辑器无法给方法提示，该对象只在运行期才拥有sayHi()方法
console.log(myClass_1.sayHi())



// obj?.prop       // 自判断静态属性访问
// obj?.[expr]     // 自判断动态访问
// func?.(...args) // 自判断函数或方法调用


//鸭子类型(Duck Typing)
//本质上就是java中的，利用接口实现多态
//ts里的接口有目前来看有两种作用：
//1、像java中一样，当作一个接口，让类来实现，从而实现多态
//2、使用鸭子类型，当作一种“模板”，模板内可以包含number、string、function等等，甚至可以
//[keyName:string]:any 这样给实现了 “模板”的对象 动态地增加属性

export interface IPoint { 
    x:number 
    y:number 
} 
export function addPoints(p1:IPoint,p2:IPoint):IPoint { 
    var x = p1.x + p2.x 
    var y = p1.y + p2.y 
    return {x:x,y:y} 
} 
 
// 正确
export var newPoint = addPoints({x:3,y:4},{x:5,y:1})  
 
// 错误 
//export var newPoint2 = addPoints({x:1},{x:4,y:3})
```
