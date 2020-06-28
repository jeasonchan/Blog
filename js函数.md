# 1 教程代码
```js
//第一种函数定义方法，函数名即为调用时的名字
function myFunction() {
    // alert(Date());  //这是页面才有的函数
    console.log(Date());
}
myFunction();


//第二种函数定义方法，通过变量名字链接到一个匿名函数，再通过“变量名”进行函数调用
//有一种c++的感觉，函数名本质上只是函数入口的指针，js中通过“函数指针”进行函数调用
var another = function (arg1) {
    console.log("输入的arg1是：" + arg1);
    console.log(Date());
}
another(123);

//甚至“指针”具有传递性，比如，下面通过简单的传递函数名的引用就实现了，不同名称指向同一函数地址
var second = another;
second(456);
var first = myFunction;
first();

//第二种函数的定义方法常常用于类的定义中！！！！


```
# 2 重点
1. 函数名本质上只是函数调用的入口的地址，这个地址可以传递给其与变量，实现不同变量指向同一个的函数入口
2. 在类方法定义中，常常使用第二种方法，将某（匿名）函数地址赋值给函数的prototype属性，实现类方法的定义
