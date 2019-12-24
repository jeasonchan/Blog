# 1背景
看前人写的代码，有一段这样设置class的模板（html）代码：[ngStyle]="setStyles()"，其中setStyle()函数的返回值是：
```js
{
	"box-shadow": "10px 10px 5px gainsboro"
}
```
其实，也可以这样写：
```html
<div [ngStyle]="{'box-shadow':isChoosen?'green':'white'}"><div/>
```
在此总结一下的[ngclass]、[ngstyle]、class、[class]、style、[style]的写法区别
# 2 代码实践展示区别
```html
<!--angular 动态绑定-->
<div [ngStyle]="{'background-color':'green'}"></div>
<div [ngStyle]="{'background-color':falg== 'default'?'green':'red'}"></div>

<div [ngClass]="{'text-success':index == 0}"></div>
<div [ngClass]="{'text-success':true}"></div>
<div [ngClass]="{'text-success':property1.isValid && property2.isValid}"></div>
<div [ngClass]="'someClass'"></div>
<div [ngClass]="getSomeClass()"></div>

<!--html 原生用法 class和style-->
<div style="color: #0c7beb"></div>
<div class="class_value"></div><!--class_value是css文件中定好的class类-->
```
```js
getSomeClass(){
	const isValid=this.property1 && this.property2;
    return {someClass1:isValid , someClass2:isValid};
}
```
# 3 总结
1. html原生的class和style没啥好总结
2. ngclass竟然也可以直接“静态”绑定，动态绑定的话，只要动态绑定的表达式的结果是{'class_name':boolean}形式的对象即可
3. ngstyle和ngclass类似，动态绑定的话，只要，返回的表达式的结果是类似一下的对象即可：
```js
{
	"box-shadow": "10px 10px 5px gainsboro",
	"color": "red"
}
```
