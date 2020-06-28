@[TOC]
# 1 代码实践
仅讨论在脱离html，纯js环境中调用别的js文件中的函数。
前提：index.js为执行入口，multi_export.js中导出了多个函数供外部使用，single_export.js导出了一个函数供使用。三个js文件的源码如下：

multi_export.js
```js
function printDate(){
    console.log(Date());
}

function printHelloWorld(){
    console.log("Hello world!");
}

module.exports={
    printHelloWorld,
    printDate
};
```

single_export.js
```js
function printDate(){
    console.log(Date());
}

module.exports=printDate;
```

index.js
```js
var moudle_multi_export=require("./multi_export.js");
moudle_multi_export.printDate();
moudle_multi_export.printHelloWorld();

var singe_export_in_moudle_single_exportJs=require("./single_export");  //require进来的js文件可以不加.js后缀
singe_export_in_moudle_single_exportJs();
```
最终成功调用了导出的函数，打印输出为：
```bash
Fri Aug 16 2019 10:08:26 GMT+0800 (GMT+08:00)
Hello world!
Fri Aug 16 2019 10:08:26 GMT+0800 (GMT+08:00)
```
# 2 感悟1.0
* js中的函数名果然是只是地址，可以随便传递给变量，加个括号就能当函数直接执行并返回结果
* 模块的导出也很直白，导出的语法很十分灵活，可以在1、module.exports={……}中定义并导出，也可以 2、在外面定义好，再导出，本文即采用先定义好，再集中导出的方法

# 3 拓展（边定义边导出）

```js
module.exports = {
    printHelloWold=function () {
        console.log("hello woeld!");
    },
    printDate=function () {
        console.log(Date());
    }
};

// 边定义边导出，本质上就是利用了，将匿名函数的入口地址传递给变量，没啥花头
```
本以为像上面一样，就能直接在index.js中调用了，但是报错“SyntaxError: Invalid shorthand property initializer”，发现，原来不能使用赋值符号将匿名函数的入口地址传给变量printHelloWold和printDate，要改用json形式的赋值方式：
```js
module.exports = {
    printHelloWold:function () {
        console.log("hello woeld!");
    },
    printDate:function () {
        console.log(Date());
    }
};

// 边定义边导出，本质上就是利用了，将匿名函数的入口地址传递给变量，没啥花头
```
# 4 感悟2.0
module.exports = {……}导出表达式，其实最标准的是json的格式，即:
```js
module.exports = {
   希望被外部看到的名字1:本js文件中的名字或者临时生成的匿名地址1,
   希望被外部看到的名字2:本js文件中的名字或者临时生成的匿名地址2,
};
```
这只是ES5标准下的写法，在vscode中进行实践时，已经提示我可以改成ES6的语法形式了。

