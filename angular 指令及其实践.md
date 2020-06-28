@[TOC]
# 1 背景
学习使用angualr material时，做了按钮菜单的demo：
```html
<input matInput [placeholder]="placeHolder" [(ngModel)]="inputValue">
```
一开始以为“matInput”是只是一个class或者style，还以为是一种新的写法……也没有在意，后来实践的时候才知道，必须这样写，才会按钮的动画效果：
```html
<mat-form-field>
  <input matInput [placeholder]="placeHolder" [(ngModel)]="inputValue">
</mat-form-field>
```
一是，在写在“mat-form-field”内部，二是，增加“matInput”，缺少一个都没有完整的动画效果。在此学习一下指令的用法。
# 2 指令简介
## 2.1 指令分类
指令（directive）是angualr中较为顶级的类，指令一共分为三类：
* 组件(Component directive)：
用于构建UI组件，继承于 Directive 类；Component可以带有HTML模板，Directive不能有模板。
* 属性指令(Attribute directive): 
用于改变组件的外观或行为，本质上就是用来修改DOM元素的外观和行为，但是不会改变DOM结构，Angular内置指令里面典型的属性型指令有ngClass、ngStyle。**如果打算封装自己的组件库，属性型指令是必备的内容。**
* 结构指令(Structural directive): 
用于动态添加或删除DOM元素来改变DOM布局，内置的常用结构型指令有ngFor、ngIf和NgSwitch。由于结构型指令会修改DOM结构，所以同一个原生或者自定义标签上面不能同时使用多个结构型指令。如果要在同一个HTML元素上面使用多个结构性指令，可以考虑加一层空的元素来嵌套，比如在外面套一层空的\<ng-container\>\</ng-container\>，或者套一层空的\<div\>。
## 2.1内置组件指令
这个就太多了……所有的标准HTML标签都在angular中进行了组件化，都算是内置的组件指令，比如 \<input\> 标签就是一个的组件（指令）。
## 2.2 内置属性指令
**ngClass**
对于当个class的绑定，很简单(学习一下<https://blog.csdn.net/it_rod/article/details/79429457#t17>)，比如：
```html
<div [class.special]="isSpecial">The class binding is special</div>
```
想在一个标签中跟同事作用/叠加几种效果时，就要使用ngClass了：
绑定常量:
```html
<div [ngClass]="{'class1': true }"></div>
```
绑定表达式:
```html
<div [ngClass]="{'class1': properity.a== 'a'}"></div>
```
绑定多个class：
```html
<div [ngClass]="{'class2': properity.a== 'a'， 'class2': true}"></div>
```
**ngStyle**
可以根据组件的状态动态设置内联样式。 NgStyle绑定可以同时设置多个**内联样式**。样式绑定<http://blog.csdn.net/it_rod/article/details/79429457#t18>是设置单一样式值的简单方式。
要同时叠加多个内联样式，可以使用NgStyle指令。
绑定常量：
```html
<div [ngStyle]="{'background-color': 'red'}"></div>
```
绑定表达式：
```html
<div [ngStyle]="{'background-color': properity.a== 'a' ? 'red' : 'green'}">
```
绑定多个style样式：
```html
<div [ngStyle]="{'background-color': properity.a== 'a' ? 'red' : 'green'， 'color': ‘black’}">
```
## 2.2 内置结构指令
<https://blog.csdn.net/it_rod/article/details/79433887>
