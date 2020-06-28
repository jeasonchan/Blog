@[TOC]
# 1 背景
使用公司前端组件库时，用到了@viewchild，一脸懵逼，场景就是通过模板引用变量获取了对应的模板实例及其对应的ts实例。经过学习，发现，@ViewChild和@ViewChildren是Angular提供给我们的装饰器，用于从模板视图中获取匹配的元素。获取模板元素的操作是在在父组件钩子方法ngAfterViewInit调用之前进行的。
# 2 基本用法
```js
@ViewChild(入参) 变量名:类名;
```
这个装饰器的直接目的，就是通过入参来获得模板背后的某个类实例，通过对获得类实例进行操作从而实现DOM操作或者其他逻辑。
入参的类型的有：1、模板引用变量名，2、public权限的类名，比如component、directive还有一些组件主动provider出来的类，3、TemplateRef，本质是还是一个类，只是第二种类型的特殊情况，组件是angular自定义的而已。根据两种情况分别举两个例子：
**入参是模板引用变量**
```html
<!--两个-->
<my-component #cmp1="myComponent"></my-component>
<my-component #cmp2模板引用变量></my-component>
```
```js
@ViewChild('cmp1') var1:myComponent;
@ViewChild('cmp2') var2:myComponent;
//一些var1和var2的操作……
```
**入参是类名**
```js
//一个模板和组件类定义在一起的一个组件
import {Component, OnInit} from '@angular/core';
import {ChildService} from './child.service';

@Component({
  selector: 'app-child',
  template: `
    <h1>自定义的一个子组件</h1>
  `,
  providers: [
    ChildService
  ]
})
export class ChildComponent implements OnInit {
  constructor(public childService: ChildService) {
  }

  ngOnInit() {
  }

}
```
```html
<app-child></app-child>
```
```js
//对html中的app-child对应的类实例和暴露出来的服务类进行使用
@ViewChild(ChildService) service:ChildService;
@ViewChild(ChildComponent ) component:ChildComponent ;
//一些对service和component的操作
```
**入参是TemplateRef**
当选择器是TemplateRef（模板引用）的时候，则会获取到html里面**所有**的ng-template类型的节点。实际例子如下：
```js
  @ViewChild(TemplateRef) template: TemplateRef<any>;
  @ViewChildren(TemplateRef) templateList: QueryList<TemplateRef<any>>;
```
但是！！！！！TemplateRef也只不过是一类特殊的组件/类而已。
## 2.1 小结
最本质的！入参直接是一个的模板引用变量名的情况下，能取到的对应的组件实例肯定只有一个，这样使用即可：
```js
@ViewChild('cmp1') var1:myComponent;var1:myComponent;
```
入参是是一个class名，比如自己定义的组件class、 组件provider出来的class、TemplateRef等等，这样使用：
```js
@ViewChild(ChildService) service:ChildService;
@ViewChildren(TemplateRef) templateList: QueryList<TemplateRef<any>>;
```
使用@ViewChild，按照类型取DOM时，如果存在多个，只取出现的第一个；想全部取到的话，就用@ViewChildren；
所以，本质上，用模板引用变量还是根据类型取DOM，根据适合的场景进行选择。
# 3 代码实践
先写了一个自定义组件，MyInputComponent，选择器是 app-my-input
```html
<p>
  my-input works!
  手动实现一个的输入框
</p>
<hr>
<mat-form-field>
  <input matInput [placeholder]="placeHolder" [(ngModel)]="inputValue">
</mat-form-field>
```
```js
import {Component, OnInit} from '@angular/core';

@Component({
  selector: 'app-my-input',
  templateUrl: './my-input.component.html',
  styleUrls: ['./my-input.component.css']
})
export class MyInputComponent implements OnInit {

  public placeHolder: string;
  public inputValue: string;

  constructor() {
  }

  ngOnInit() {
    this.placeHolder = 'Default Holder';
  }

}

```
暴露出两个的属性，让父组件进行读取、使用。
在父组件中对这个自定义组件进行调用：
```html
<hr>
<span>模板引用变量 input1</span>
  <app-my-input #input1></app-my-input><br>
  <span>从本组件的属性中读取的值为：{{input1_in_main_component.inputValue}}</span>
<hr>
<span>模板引用变量 input2</span><br>
<app-my-input #input2></app-my-input>
<span>直接从模板引用变量中读取的值为：{{input2.inputValue}}</span>
<hr>
<span>模板引用变量 input3</span>
<app-my-input #input3></app-my-input>

<ol>
  <li *ngFor="let eachInput of inputList;">
    {{eachInput.placeHolder}}<br>{{eachInput.inputValue}}
  </li>
</ol>
```
```js
@ViewChild('input1') input1_in_main_component: MyInputComponent;
@ViewChildren(MyInputComponent) inputList: MyInputComponent[];
```
想通过viewchild的方式访问模板引用变量input1的属性值，并且对得到的实例进行调用，在页面上进行显示。
对于模板引用变量input2，可以直接在页面上访问、使用其属性值。
同时，通过ViewChildren实现了对模板页面上MyInputComponent组件的DOM选取，成功读取了自定义组件中的属性placeholder和inputValue的值。
最终，实现效果如下：
![实现图](https://img-blog.csdnimg.cn/20191024233250621.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2plYXNvbl9jaGFuX3pqdQ==,size_16,color_FFFFFF,t_70)
