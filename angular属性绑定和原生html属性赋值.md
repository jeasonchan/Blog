@[TOC]
# 1 angualr中属性绑定
根据数据绑定的方向和绑定方式共有四种：
1. 使用html的原生绑定方式
```html
<div title="titleNameForDiv">
  <!-- 不用中括号对 -->
  鼠标放到我这里查看本div的title
</div>
```
原生方式，只支持绑定固定值，因此，就算titleNameForDiv这个变量在angular中的相应的ts文件定义了，这个div的 title也只能是"titleNameForDiv"这个字符串，并不是这个变量的值。
实在想用原生的属性，从ts取值，则要改为这样：
```html
<div title={{titleNameForDiv}}>
  <!-- 不用中括号对 -->
  鼠标放到我这里查看本div的title
</div>
```
不建议这种写法，不够纯粹，建议使用纯粹的angular单向绑定。
2. 从ts到模板的单向绑定
```html
<div [title]="titleNameForDiv">
  <!-- 不用中括号对 -->
  鼠标放到我这里查看本div的title
</div>
```
数据的流向为，ts中的属性到html（模板文件）。
这种数据流向的单向绑定，常用的还有class和style的单向绑定。
3. 从模板到ts的单向绑定
```html
<button (click)="clearScreen()">clear</button>
```
4. 双向绑定
模板的数据和ts中的属性的值，实时匹配更新，比如：
```html
<p>
  {{creator}}
</p>

<input type="text" name="输入框" [(ngModel)]="creator">
```
在input中进行输入的过程中，上面的段落标签中的内容会和input输入框实时一致。
# 2 html属性赋值
纯粹的html只能从html文件中的js脚本中取值，如：
```html
<button  onclick="clearScreen()">clear</button>
```
如果在angular工程中使用以上写法，注定报错，因为，button 中不能使用 onclick="function()"  这格式写法是html中button原生的属性，浏览器会试图在html中的js脚本中寻找function()这个函数， 然而，因为html连js都没有，注定会报 function 未定义错误。
# 3 异同
唯一的相同点就是，都是为了给html更新数据，不同的是，更新的方式不同，angular的基于虚拟DOM的模板操作比原生强大的多。
