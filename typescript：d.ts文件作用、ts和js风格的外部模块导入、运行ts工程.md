@[TOC]
# 1背景
想给妹子写一个导入的导入excel中的内容并自动填充到网页中的脚本，脚本通过油猴实现的运行，所以，第一步就是在现在本地实现excel内容的解析。因此，在本地运行时使用nodejs作为运行环境，由于和js或者ts相关的学习过程只有简单的angular经历，纯nodejs后台、本地项目没有经验，特来记录一下自己的学习过程，以及强烈谴责复制粘贴的博客行为，完全是对一些新手的误导！！！！！
接下来，以从本地读取excel文件并打印处详细为例，记录注意点和要点。
# 2 从零开始示范读取本地excel文件
1. 新建工程
直接npm init或者新建一个文件夹都可以。
2. 创建程序入口文件
使用npm init时，会自动问你入口文件，比如index.js，这个文件就可以作为你的工程入口文件，可以自己在package.json里面改，影响不大，创建Main.ts文件作为项目的入口文件；自己创建文件夹时，可以创建叫Main.ts。注意，是ts文件。
3. 先写个hello world实验一下。
Main.ts
```js
console.log("Hello world!");
```
由于ts是没有的办法直接运行的，都必须先编译成js，再通过v8等脚本引擎运行，比如chrome、nodejs等等。
cd到项目目录下，运行"tsc Main.ts"，这一步，ts编译器的会将Main.ts所涉及的ts文件都编译成js文件，比如自己import进来的ts文件里的类；运行完会发现多了一个Main.js文件，在"node Main.js"，就能在控制台打印出内容了。
注：tsc需要自己安装，使用类似于npm install -g typescript@2.9.2的命令进行全局安装，不加版本号的话会默认安装最新的版本。根据自己的情况安装相应的版本。

4. 导入自己需要的类
在ts里导入类其实很方便！！几乎跟java一毛一样，先来看看导入自己写的工具类。
工具类CommonUtil.ts
```js
export class CommonUtil{
	public static printHello(){
		console.log("Helllo world!");
	}	
}
```
Main.ts
```js
//导入
import { CommonUtil } from './CommonUtil';
//调用
CommonUtil.printHello();
```
但是！到多时候，我们都需要去使用的npm中庞大、好用的js脚本，比如本文中要使用的xlsx；先使用"npm install --save xlsx"将相关资源脚本安装到node_modules文件中，并将这个xlsx依赖写进package.json中。
走到这一步，其实已经可以使用这个，使用方法就是:
```js
let Xlsx = require('xlsx');
//不用let应用const其实更好！
```
而且，去百度typescript导入xxx时，大部分都是教你用这种方法，但是！这其实时js里的用法，在ts也可以用，但是真的不纯粹，不现代化，不严格，还是希望能像前文导入自己的方法那样导入js工具类。
这时候，就可以是使用"npm install @types/xlsx"（同样的，还可以安装其他js模块的ts转换模块，比如@types/express），安装的xlsx的xxx.d.ts及相关文件，安装的这个模块其实就相当于的将js的模块和ts模块之间的转换模块，就像使用angular-material时一般都会自己建一个的material module，导入所有的材料模块，然后再都导出来给app module使用。
同时，可以发现，"npm install @name"是安装ts的包，而包名前不加@，比如"npm install name"，就是安装js的包，这点十分重要。
安装完的xlsx模块的ts声明模块，开始导入使用：
```js
import * as Xlsx from 'xlsx'
//新建一个Xlsx.WorkBook类的实例
let excelFile: Xlsx.WorkBook = Xlsx.readFile("../tampermonkey-scriptps/resource/新建 Microsoft Excel 工作表.xlsx");
```
导入xlsx其实是到了xlsx这个命名空间/模块，并在自己的程序里将其重命名为Xlsx，并且可以发现Xlsx命名空间里，不及包含类，还包含了函数、变量等。
通过这样统一、整齐的import方法，ts看上去整齐多了！

5. 接下来就是对Xlsx中资源的使用，基本都能做到命名即文档，很简单易懂，不做赘述。最后，想查看运行效果就先编译为js文件，在用node运行同名的js文件即可。

# 3 总结
xxx.d.ts的模块/命名空间声明文件很重要，基本就相当于这个模块的文档，可以多看看。
研究ts的模块导入时，看了很多博文，基本都是复制粘贴的，坑的一批……在此处强烈谴责！

