js中的任何function都可以new，看上去每个方法都像java或者c++中的构造函数。不过，java和c++对构造函数进行new操作得到的是，和构造函数同名的类的实例（但是，构造函数并显式的返回值和void。）
那么js的中对任意一个function进行new操作是做的什么事情？
```js
var Func=function(){};
var func=new Func ();//对这一步进行步骤分析
```
new共经了四个阶段： 
```js
var obj=new Object(); //1 隐式地创建一个空对象
obj.__proto__= Func.prototype;  //2 设置原型链，因此，只有new出来的实例对象才有prototype属性，类本身不包含prototype属性
var result =Func.call(obj);  //3 让Func中的this指向obj（这一小步没写出来），并执行Func的函数体。
if (typeof(result) == "Object"){  //4 判断Func的返回值类型，如果是值类型，返回obj。如果是引用类型，就返回这个引用类型的对象。
  func=result;
}else{
  func=obj;
}
```
还是要深刻的读一读<https://www.cnblogs.com/pizitai/p/6427433.html>
