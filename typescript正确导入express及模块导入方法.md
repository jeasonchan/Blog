# 1 背景
刚接触express时，看网上的教程，几乎清一色的使用：
```js
var app=require("express");
app.get(......)
```
**SHIT！！！！！简直人云亦云，毫无灵魂！！！！** 和其他模块的导入风格放在一起，简直难受。
先来看看我认为的最正确，最typescript的使用方式：

```js
import { Express, Application } from "express";  //本质上是导入了 namespace e 中的接口
import newServer = require("express");  //使用了export=e 这种的导出方式
var server: Express = newServer();
```
简直是最有灵魂、最typescript的导入方法，不服来辩！
# 2 为什么要这样导入？
先看一下express的导出/定义文件：**index.d.ts**
```js
declare function e(): core.Express;

declare namespace e {
	//xxxxxx
}
export=e
```
对应于"export=e"，这种方式导入的是只是一个函数名 e （实际上不只是一个函数名，就模糊过去吧……），调用该函数e返回core.Express类型的对象，如果该给这个变量声明类型，就一点也不typescript，也没有各种类型提示了……所以接着从express中导入各种接口/类 import { Express, Application } from "express"; 
# 3 为什么要两种不同风格的方式导入？
先来介绍一些ts中的模块。
**namespace_exercise_2.ts**
```js
//不期望该命名空间导出，不加export
namespace MyNameSpace {
    export var ArgInOtherFile: string = "hahah";

}

//期望该命名空间导出，加export
export namespace MyNameSpace_2 {
    export function printHello(): void {
        console.log("Hello world!")
    }
    function test():void{
        
    }
}

//一定要严格遵守！！！
//希望在别的文件中使用的就用export！！！
//不希望的就不加！！！
export var hahahah: string = "aaaa";
```

**Main.ts**
```js
/*
模块是在其自身的作用域里执行，并不是在全局作用域，
这意味着定义在模块里面的变量、函数和类等在模块外部是不可见的，
除非明确地使用 export 导出它们。
类似地，我们必须通过 import 导入其他模块导出的变量、函数、类等。

两个模块之间的关系是通过在文件级别上使用 import 和 export 建立的。

模块使用模块加载器去导入其它的模块。 
在运行时，模块加载器的作用是在执行此模块代码前去查找并执行这个模块的所有依赖。
JavaScript模块加载器是服务于 Node.js 的 CommonJS 和
服务于 Web 应用的 Require.js。此外还有有 SystemJs 和 Webpack。
*/

//导入别的文件中的所有模块
//也可以直接用这种方式导入别的文件中的命名空间  另一种是  /// <reference path="./xxxxx"/>>
import * as AnotherFile from "./namespace_exercise_2";
AnotherFile.MyNameSpace_2.printHello();

//导入别的文件中的单个模块
import { hahahah } from "./namespace_exercise_2";
console.log(hahahah);

/// <reference path="./namespace_exercise_2"/>
//完全等效于
import "./namespace_exercise_2"
//使用时就直接使用了，有可能会带来导入的模块/文件的重名问题

/*
用不同的加载引擎导入模块
tsc --module commonjs TestShape.ts
tsc --module amd TestShape.ts

tsconfig.json有配置项，ts默认配置的是commonjs
*/
```
所以，express为了兼容性，使用了两种风格的导出，只有使用了ts风格的import导入才能实现ts中的强类型。
