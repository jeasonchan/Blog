@[TOC]
# 基础
网址组成：主机地址+path+request+hash

浏览器调试：
F12，consle标签，consle.log(xxxx)，可用来调试，注意同行右侧的数字，表示执行此输出的行号
换行，不标准使用<b/>，，标准使用<div>包一下。

开发中全局的元素才会用id，比较“重型”的操作，一般的都是使用name来进行索引。

HTML DOM：html的文件对象模型
大神代码
```html
<!DOCTYPE html>

<head>
    <meta charset="UTF-8">
    <title>大佬的名单录入页面</title>

    <style type="text/css">
        a{text-decoration:none;}
        div{margin:0px auto;height:25px;width:720px;}
        table{width: 500px;border: 1px solid gray;border-collapse: collapse;margin:50px auto;}
        th,td{line-height: 35px;border: 1px solid gray;text-align: center;}
    </style>

</head>

<body>
<div>  <!--属性值使用双引号-->
    <div><label for="labelName" >姓名：</label> <input type="text" id="labelName"></div>
    <div><label for="labelAge">年龄：</label> <input type="text" id="labelAge"></div>
    <div><label for="labelAddress">住址：</label> <input type="text" id="labelAddress"></div>
    <div><button id="buttonAdd" >添加</button></div>
</div>
<br>
</body>

<script type="text/javascript">
    //DOM的基本对象是node，代表着一个元素实例对象

    //查找节点
    var VarLabelName=document.getElementById('labelName');//获取元素name或者id使用单引号
    var VarLabelAge=document.getElementById('labelAge');
    var VarLabelAddress=document.getElementById('labelAddress');
    var VarButtonAdd=document.getElementById('buttonAdd');
    //创建table
    var VarNewTable=document.createElement('table');
    //创建tr，创建表格其中的一行用于稍后的表格赋值
    //这个其实是表格首行
    var tr=gettr('姓名','年龄','住址','操作',true);

    //添加点击事件
    VarButtonAdd.onclick=clickEvent;//代码层面没有复用，可以直接用匿名函数

    //点击事件函数
    function clickEvent(){
        //判断输入框是否为空
        if(VarLabelName.value==''||VarLabelAddress.value==''||VarLabelAddress.value==''){
            return;//其中任意一个为空都不执行
        }else{
            document.body.appendChild(VarNewTable);//将table追加到body中，默认加到末尾
            var tr=gettr(VarLabelName.value,VarLabelAge.value,VarLabelAddress.value,'删除',false);//输入内容
            //清除输入框内容,输入框中的字消失
            VarLabelName.value='';
            VarLabelAge.value='';
            VarLabelAddress.value='';
        }
    }

    //创建行
    //输入参数的含义：姓名，年龄，住址，操作，布尔值，元素类的名称
    function gettr(name,age,www,cz,bool){
        var otr=document.createElement('tr');//创建的元素种类名称，返回的是对应元素的一个node
        //创建列
        if(bool){
            var oth1=getth(name,false,'th');
            var oth2=getth(age,false,'th');
            var oth3=getth(www,false,'th');
            var oth4=getth(cz,false,'th');
        }else{
            var oth1=getth(name,false,'td');
            var oth2=getth(age,false,'td');
            var oth3=getth(www,false,'td');
            var oth4=getth(cz,true,'td');//表示当前的表格是删除按钮
        }
        //将列追加到行中
        otr.appendChild(oth1);
        otr.appendChild(oth2);
        otr.appendChild(oth3);
        otr.appendChild(oth4);

        VarNewTable.appendChild(otr);//将tr追加到table中
        return otr;
    }

    //创建单行的单列，其实就是创建一个单元格
    function getth(content,sl,elementClass){
        var oth=document.createElement(elementClass);
        if (sl) {//表示是否为删除操作按钮
            //创建a链接
            var aa=document.createElement('a');
            aa.innerHTML=content;//为a链接添加内容
            aa.href='#';//为链接添加属性
            //为链接添加点击事件
            aa.onclick=function(){
                if(confirm('是否删除?')){//判断是否删除
                    var tth=this.parentNode.parentNode;//this这里指这个超链接
                    tth.parentNode.removeChild(tth);
                }
            }
            oth.appendChild(aa);//将a链接追加到列中
        }else{
            oth.innerHTML=content;//为列附加内容
        }
        return oth;//返回值到列
    }
</script>


</html>
```
我的代码
```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8">
    <title>名单录入系统</title>
</head>

<body>
<h1>名单录入系统</h1>
<p>姓名：<input type="text" id="name"></p>
<p>工号：<input type="number" id="workId"></p>
<p>职务：
    <select id="job">
        <option value=""></option>  <!--用于显示空白-->
        <option value="经理">经理</option>
        <option value="技术Leader">技术Leader</option>
        <option value="开发">开发</option>
    </select>
</p>
<p><button id="sureButton">录入</button></p>
<table id="myTable">
    <tr> <!--表示一行-->
        <th>姓名</th><!--th表格标题列，在tr中的列，其实就是一个单元格-->
        <th>职务</th>
        <th>工号</th>
        <th>操作</th>
    </tr>
</table>
</body>

<script>
    //绑定变量和节点
    var nameInput=document.getElementById('name');
    var workIdInput=document.getElementById('workId');
    var jobSelect=document.getElementById('job');
    var sureButton=document.getElementById('sureButton');
    var mytable=document.getElementById('myTable');


    //为点击按钮添加事件
    sureButton.onclick=function(){
        //先从输入框和多选框构建出一行
        var myNewTr=newTr(nameInput.value,workIdInput.value,jobSelect.value);

        //将一行添加至表格中
        mytable.appendChild(myNewTr);

        //将输入框中的内容清除
        nameInput.value='';
        workIdInput.value='';
        jobSelect.value='';
    }

    //
    function newTr(name,job,workId) {
        var myNewTr=document.createElement('tr');

        var tdName=document.createElement('td');
        tdName.innerHTML=name;
        var tdJob=document.createElement('td');
        tdJob.innerHTML=job;
        var tdworkId=document.createElement('td');
        tdworkId.innerHTML=workId;
        var deleteAction=document.createElement('button');
        deleteAction.innerHTML="删除";

        myNewTr.appendChild(tdName);
        myNewTr.appendChild(tdJob);
        myNewTr.appendChild(tdworkId);
        myNewTr.appendChild(deleteAction);
        console.log("hahha");
        return myNewTr;
    }

    //
    function deleteAtr(){

    }





</script>
</html>
```





# Angular

## npm
NPM是随同NodeJS一起安装的包管理工具，能解决NodeJS代码部署上的很多问题，常见的使用场景有以下几种：
* 允许用户从NPM服务器下载别人编写的第三方包到本地使用。
* 允许用户从NPM服务器下载并安装别人编写的命令行程序到本地使用。
* 允许用户将自己编写的包或命令行程序上传到NPM服务器供别人使用

## 安装angular
由于网络环境不佳，直接修改npmrc文件，追加仓库描述，使用阿里的npm镜像：
```
registry=https://registry.npm.taobao.org/
```
npm install -g @angular/cli@6.2.9   //全局安装后，会将angular的**ng**命令名称添加至系统中；有些项目特有的不用加-g，可以加--save，以防以后会复用

或者使用类似于npm config set registry http://registry.npmjs.org  的命令进行安装

## 项目结构
单文件的html分开为模块（module）化，每个module中有许多component，module负责导入别的module给本module的component用，component还需要再import自己真正需要使用的component。component真正能import的第三方包只限于自己所在的module所导入的。

元数据注解声明当前模块或者当前组件的各种信息，十分有用！

## 杂七杂八和注意事项
* 元素，组件，元素就是html标准语法里的元素，比如，body、h1什么的，自己定义的组件（component）时，元数据里包含了当前组件的selector名称，相当于元素的名称，因此，组件用起来就跟元素差不多，完全就是自定义的元素。
* 指令，ngClass，
* CSS类
* 样式
* 要想组件的property能够在被父组件调用时，能够实现property绑定，则组件的这个要被绑定的property必须有@Input()注解！这个**是父组件和字组件进行数据交互的重要方式！**也正是通过这种方式，我们实现了在传统的html中对元素的attribute进行设置。
```html
<my-component [myProperty]="233333"></my-component>
```
* 组件想使用A serivice时，依然要在当前组件ts中定义一个同class的成员变量，主要还是用来存储 指向系统自动创建出来的seivice 单一实例 的指针，方便在方法中调用这个服务，使用方法的

##	模板语句和模板表达式
常见的语法及感悟如下：
```html
<!-- "The sum of 1 + 1 is 2" -->
<p>The sum of 1 + 1 is {{1 + 1}}.</p>

<!--遍历数组生成多个宿主元素-->
<ul>
  <li *ngFor="let customer of customers">{{customer.name}}</li>
</ul>

<!--对当前元素起名，并进行调用，此处的调用使用了angualr的插值-->
<label>Type something:
  <input #customerInput>{{customerInput.value}}
</label>

<!--圆括号为事件绑定，触发双引号中的函数-->
<button (click)="deleteHero()">Delete hero</button>
```

## 绑定
绑定的类型可以根据数据流的方向分成三类： 从数据源到视图、从视图到数据源以及双向的从视图到数据源再到视图。
* 从数据源（组件/ts）到视图（html/模板），就是把等号右边的“赋值”给等号左边的，用法有：插值、Attribute、CSS 类样式

```html
{{expression}}
[target]="expression" 
<!--将ts中的expression的值绑定给当前元素/组件的target property，从而可以通过ts操作当前元素的property-->
<!--再配合该元素的是事件绑定，轻松实现对该元素所包含的内容进行更改！-->
bind-target="expression"
```

* 从视图（html/模板）到数据源（组件/ts），左边“赋值”给右边，用法只有事件绑定
```html
(target)="statement"  
on-target="statement"  <!--html语法，上面是angular语法-->
```

* 双向绑定，意味着双向绑定用到的宿主元素，既能显式，有能输入值，比如，<input>元素

```html
[(target)]="expression"
bindon-target="expression"
```

## 模板attribute和组件property
angular中，每个组件至少对应三个文件：模板文件、组件（脚本）、样式文件
* 模板即为html文件，即为“用户界面”，为了避免恶意脚本，angualr直接忽略html页面中的脚本，原始html元素（比如，<body>）的属性，一般叫做attribute
* 组件（脚本），即为每个为组件/component的ts文件，ts主要分为三部分：导入模块声明、元数据声明、类文件定义，这个里面定义的组件的属性叫做property
*  样式文件，即为css文件

使用的时候，一个组件看上就像一个元素，“嵌入式”的在一个html中使用。

元素里不加括号的属性：一般就是html的attribute
圆括号属性：事件绑定、由模板向组建的数据绑定
方括号属性：由组件向模板的数据绑定

attribute 是由 HTML 定义的。property 是由 DOM (Document Object Model) 定义的。

少量 HTML attribute 和 property 之间有着 1:1 的映射，如 id。有些 HTML attribute 没有对应的 property，如 colspan。有些 DOM property 没有对应的 attribute，如 textContent。

attribute 初始化 DOM property，然后它们的任务就完成了。property 的值可以改变；attribute 的值不能改变，所以说纯html的网页是静态网页。

例如，当浏览器渲染 <input type="text" value="Bob"> 时，它将创建相应 DOM 节点， 它的 value 这个 property 被初始化为 “Bob”。

当用户在输入框中输入 “Sally” 时，DOM 元素的 value 这个 property 变成了 “Sally”。 但是该 HTML 的 value 这个 attribute 保持不变。如果你在浏览器开发者模式读取 input 元素的 attribute，就会发现确实没变： input.getAttribute('value') // 返回 "Bob"。

HTML 的 value 这个 attribute 指定了初始值；DOM 的 value 这个 property 是当前值。

disabled 这个 attribute 是另一种特例。button的 disabled 这个 property默认情况下 是 false，所以button默认是可用的。 当你添加 disabled 这个 attribute 时，只要它出现了按钮的 disabled 这个 property 就初始化为 true，于是按钮就被禁用了。所以，attribute 的值无关紧要，有无这个attribute决定了button是否可用，或者，直接完全使用property即可。

## 模块划分与依赖注入思想
先看一段典型的应用场景代码：
```javascript
@NgModule({
  declarations: [
    AppComponent,
    ItemDirective
  ],
  imports: [
    BrowserModule,
    FormsModule,
    HttpModule
  ],
  providers: [],
  bootstrap: [AppComponent]
})
export class AppModule { }
```
使用了ngmodule注解了，表示当前文件定义的是一个module，并且注解中的参数，也就是变量，有：
* imports: 导入其它带有组件、指令和管道的模块，这些模块中的元件都是本模块所需的。和java一样，import操作并不具有继承性！
* declarations: 声明某些组件、指令和管道属于本模块。
* exports: 公开本模块中的部分组件、指令和管道，以便其它模块中的组件模板中可以使用它们。可以看出，exports的组件一定是小于等于declaration的，并且参数后面的东西都是以数组/集合的形式罗列出
* providers: 声明本模块中的服务，服务不需要导出，因为服务是全局单例的，一旦被初始化，就已经全局通用了，相反如果重复的导入提供了同一服务的模块就可能发生问题；对于服务更好的做法是写一个核心模块，专门提供全局服务，保证此模块只会被根模块引用一次，然后所有的子组件就都已经可以使用这些全局服务了。

模块间的共享组件、共享服务、懒加载下的服务共享：<https://segmentfault.com/a/1190000014067139>

浅谈angular中的模块结构：<https://www.cnblogs.com/yitim/p/angular2-study-module-framework.html>

## 新建项目
ng new firstAngular  //使用angular命令新建一个项目

cd firstAngular   //cd 进入项目文件夹

ng serve  //已调试模式启动当前所在文件夹项目，支持热修改

有时候单个项目需要安装特定的依赖包，在项目所在文件进行 npm install --save xxxxx@xxxxxx 安装

SPA：单页面

app-root：

 MVVM：M+VM+V
 angular使用MVVM模型，中间层的VM负责将数据层的Model和视图层的View
更新的方向， 父→子，比如，@input
更新的方向， 子→父，异步（让别人去做，被人做完后通知自己），比如，@onput

路由，route，使SPA应用看上去像是多页面
# 调试运行发布
将ts脚本转译成js脚本，然后在根目录下生成dist目录。href /iui/chenrui  这个属性被写在dist目录中的index.html中，其实就是网址中的path路径
```
"build": "ng build --prod --base-href /iui/chenrui/"   最后的斜杠一定不能少！！！
```
使用本local服务器调试，因为浏览器的安全机制，必须使用代理连接至local服务器
"local":"ng serve --proxy-config proxy.conf.json"

# node.js后端
纯JavaScript写的后端，为轻量级的后端，js和ts较为接近，比较简单。

启动后端，node xxxx.js，启动这个脚本从而启动服务器，和同为脚本语言Python的极为相似。


