# 1 背景
之前一直使用typescript，没怎么接触过用于模块、文件导入的require函数，最近在用油猴写一个的脚本，有一个十分迫切的需求：油猴脚本中所有的js文件都写在一个函数中，typescript如果引入其他模块，要么使用“\<script src="https://cdn.bootcss.com/xlsx/0.15.1/cpexcel.js"\>\</script\>”这样的方式从网络中读取脚本，要么在本地使用“tsc --out main.js main.ts”的方式将所有用到的代码编译进一个js文件，但是要想编译进一个文件，则必须实用AMD（异步加载依赖）的方式，想要异步加载依赖又要使用require函数，不仅就有疑问：
commonjs也是用的rquire，require究竟是干什么的？commonjs、exModule、AMD这些模块引擎有什么区别？ts项目使用不同的模块引擎时，该怎样导入导出模块？

# 2 commonjs、esModule和AMD、CMDS的区别和联系
在JavaScript发展初期就是为了实现简单的页面交互逻辑，寥寥数语即可；如今CPU、浏览器性能得到了极大的提升，很多页面逻辑迁移到了客户端（表单验证等），随着web2.0时代的到来，Ajax技术得到广泛应用，jQuery等前端库层出不穷，前端代码日益膨胀，这时候JavaScript作为嵌入式的脚本语言的定位动摇了，JavaScript却没有为组织代码提供任何明显帮助，甚至没有类的概念，更不用说模块（module）了，JavaScript极其简单的代码组织规范不足以驾驭如此庞大规模的代码。

## 2.1 模块的由来
既然JavaScript不能handle如此大规模的代码，我们可以借鉴一下其它语言是怎么处理大规模程序设计的，在Java中有一个重要带概念——package，逻辑上相关的代码组织到同一个包内，包内是一个相对独立的王国，不用担心命名冲突什么的，那么外部如果使用呢？直接import对应的package即可，比如：
```java
import java.util.ArrayList; 
```
遗憾的是JavaScript在设计时定位原因，没有提供类似的功能，开发者需要模拟出类似的功能，来隔离、组织复杂的JavaScript代码，称为模块化。一个模块就是实现特定功能的文件，有了模块，我们就可以更方便地使用别人的代码，想要什么功能，就加载什么模块。模块开发需要遵循一定的规范，各行其是就都乱套了，规范形成的过程是痛苦的，可以把所有的模块成员封装在一个对象中
```js
var myModule = {
    var1: 1,

    var2: 2,

    fn1: function(){

    },

    fn2: function(){

    }
}
```
这样我们在希望调用模块的时候引用对应文件，然后，myModule.fn2(); 这样避免了变量污染，只要保证模块名唯一即可，同时同一模块内的成员也有了关系，看似不错的解决方案，但是也有缺陷，外部可以随意修改内部成员，比如：myModel.var1 = 100; 这样就会产生意外的安全问题，我偷偷修改内部的变量/对象，会让同一进程中的调用跟着出错，因此出现了：

**立即执行函数**，可以通过立即执行函数，来达到隐藏细节的目的：
```js
var myModule = (function(){
    var var1 = 1;
    var var2 = 2;

    function fn1(){

    }

    function fn2(){

    }

    return {
        var1：1，
        var2：2，
        fn1: fn1,
        fn2: fn2
    };
})();
```
返回的对象是和原始库的一份拷贝，这样在模块外部无法修改我们原始的变量、函数。上述做法就是我们模块化的基础，导出的也就是这个“myModule”变量。目前，通行的JavaScript模块规范主要有两种：CommonJS和AMD。

## 2.2 CommonJS规范

我们先从CommonJS谈起，因为在实际余运行的网页端没有模块化编程，只是JavaScript在运行。但是，在服务器端却一定要有模块，所以虽然JavaScript在web端发展这么多年，第一个流行的模块化规范却由服务器端（nodejs）的JavaScript应用带来，CommonJS规范是由NodeJS发扬光大，这标志着JavaScript模块化编程正式登上舞台。

1、定义模块 
根据CommonJS规范，一个单独的文件就是一个模块。每一个模块都是一个单独的作用域，也就是说，在该模块内部定义的变量，无法被其他模块读取，除非定义为global对象的属性

2、模块输出： 
模块**只有一个出口**，module.exports对象，我们需要把模块希望输出的内容放入该对象

3、加载模块： 
加载模块使用require方法，该方法读取一个文件并执行，返回文件内部的module.exports对象该对象中，有变量、有函数对象等等

看个例子：
```js
//模块定义 myModel.js

var name = 'Byron';

function printName(){
    console.log(name);
}

function printFullName(firstName){
    console.log(firstName + name);
}

//导出了两个“东西”，两个函数对象
module.exports = {
    printName: printName,
    printFullName: printFullName
}
```
在别的文件中使用别的模块中的导出对象：
```js
//加载模块
var nameModule = require('./myModel.js');

nameModule.printName();
```
所以，express库中，export=e，表明了，导出的对象是e这个函数对象，而不是e这个命名空间，所以使用js时，使用const generateExpressServer=rqurie("express")，var server=generateExpressServer()，创建服务器实例对象。

不同的实现对require时的路径有不同要求，一般情况可以省略js拓展名，可以使用相对路径，也可以使用绝对路径，甚至可以省略路径直接使用模块名（前提是该模块是系统内置模块），上面的代码，会发现require是同步的。模块系统需要同步读取模块文件内容，并编译执行以得到模块接口。这在服务器端实现很简单，也很自然，然而， 想在浏览器端实现问题却很多。浏览器端，加载JavaScript最佳、最容易的方式是在document中插入script 标签。但脚本标签天生异步，传统CommonJS模块在浏览器环境中无法正常加载。

解决思路之一是，开发一个服务器端组件，对模块代码作静态分析，将模块与它的依赖列表一起返回给浏览器端，也就是将页面js逻辑脚本和其依赖的js，打包在一起返回给浏览器。 另一种解决思路是，用一套标准模板来封装模块定义，但是对于模块应该怎么定义和怎么加载，又产生的分歧：AMD和CMD。

因此，AMD和CMD可以简单认为，commonjs的require函数的“重载”，但并不是想的那么简单。
### 2.2.1 AMD模块载入方案
AMD 即Asynchronous Module Definition，中文名是异步模块定义的意思。它是一个在浏览器端模块化开发的规范，由于不是JavaScript原生支持，使用AMD规范进行页面开发需要用到对应的库函数，也就是大名鼎鼎RequireJS，实际上AMD 是 RequireJS 在推广过程中对模块定义的规范化的产出。

requireJS主要解决两个问题：

1、多个js文件可能有依赖关系，被依赖的文件需要早于依赖它的文件加载到浏览器 
2、js加载的时候浏览器会停止页面渲染，加载文件越多，页面失去响应时间越长 
看一个使用requireJS的例子
```js
// 定义模块 myModule.js
define(['dependency'], function(){
    var name = 'Byron';
    function printName(){
        console.log(name);
    }

    //和commonjs的module.exports={xxxxx}略有不同，但总体上都是导出一个对象
    return {
        printName: printName
    };
});
```
```js
// 加载模块，一般是写在页面的\<script\>中
require(['myModule'], function (exported_Instance){
　 exported_Instance.printName();
});
```
定义/导出模块：

requireJS定义了一个函数 define，它是全局变量，用来定义模块，define(id?, dependencies?, factory)：

id：可选参数，用来定义模块的标识，如果没有提供该参数，脚本文件名（去掉拓展名）

dependencies：是一个当前模块依赖的模块名称数组

factory：工厂方法，模块初始化要执行的函数或对象。如果为函数，它应该只被执行一次。如果是对象，此对象应该为模块的输出值 

在页面上使用require函数加载模块：

require([dependencies], function(){}); 

require()函数接受两个参数，第一个参数是一个数组，表示所依赖的模块；第二个参数是一个回调函数，当前面指定的模块都加载成功后，它将被调用。加载的模块会以参数形式传入该函数，从而在回调函数内部就可以使用这些模块。

require()函数在加载依赖的函数的时候是异步加载的，这样浏览器不会失去响应（可以先加载静态页面出来，可以上线滚动页面，但是不响应点击等操作，如果是同步加载，先加载js脚本就要卡住很久了），它指定的回调函数，只有前面的模块都加载成功后，才会运行，解决了依赖性的问题。

### 2.2.2 CMD模块载入方案
CMD 即Common Module Definition通用模块定义，CMD规范是国内发展出来的，就像AMD有个requireJS，CMD有个浏览器的实现SeaJS，SeaJS要解决的问题和requireJS一样，只不过在模块定义方式和模块加载（可以说运行、解析）时机上有所不同。

AMD与CMD区别，最明显的区别就是在模块定义时对依赖的处理不同：

1、AMD推崇依赖前置，在定义模块的时候就要声明其依赖的模块 

2、CMD推崇就近依赖，只有在用到某个模块的时候再去require ，这种区别各有优劣，只是语法上的差距，而且requireJS和SeaJS都支持对方的写法，AMD和CMD最大的区别是对依赖模块的执行时机处理不同，注意不是加载的时机或者方式不同，很多人说requireJS是异步加载模块，SeaJS是同步加载模块，这么理解实际上是不准确的，其实加载模块都是异步的，只不过AMD依赖前置，js可以方便知道依赖模块是谁，立即加载，而CMD就近依赖，需要使用把模块变为字符串解析一遍才知道依赖了那些模块，这也是很多人诟病CMD的一点，牺牲性能来带来开发的便利性，实际上解析模块用的时间短到可以忽略

为什么我们说两个的区别是依赖模块执行时机不同，为什么很多人认为ADM是异步的，CMD是同步的（除了名字的原因。。。）

同样都是异步加载模块，AMD在加载模块完成后就会执行改模块，所有模块都加载执行完后会进入require的回调函数，执行主逻辑，这样的效果就是依赖模块的执行顺序和书写顺序不一定一致，看网络速度，哪个先下载下来，哪个先执行，但是主逻辑一定在所有依赖加载完成后才执行

CMD加载完某个依赖模块后并不执行，只是下载而已，在所有依赖模块加载完成后进入主逻辑，遇到require语句的时候才执行对应的模块，这样模块的执行顺序和书写顺序是完全一致的

这也是很多人说AMD用户体验好，因为没有延迟，依赖模块提前执行了，CMD性能好，因为只有用户需要的时候才执行的原因。
## 2.3 ES Module规范
commonjs是在js中概念，ES Module个人认为是存在于ts中的概念，CommonJS的模块是对象，ES Module的模块不是对象，是使用export显示指定输出的多个变量、对象等等，需要通过import输入。ES Module是存在于ts中的，此法为编译时加载，编译完生成的js就被转换成commonjs规范了。

CommonJS 模块输出的是一个值的拷贝，ES6 模块输出的是值的引用。CommonJS 模块是运行时加载，ES6 模块是编译时输出接口。

ES Module规范，经过编译，最终的归宿还是commonjs规范。

## 3 commonjs规范和ES Module规范的导入导出实践
**牢记：commonjs导出，只能是一个对象，导入导出方式相对固定；ES Module是具体的多个被export的，可按需导入，也一下子全部导入，导入导出方式相对灵活。**

参考了<https://blog.csdn.net/weixin_33696106/article/details/91428247>。



## 4 commonjs的require函数导入机制探究
参考了<https://blog.csdn.net/weixin_33901843/article/details/91425717>。


