@[TOC]
# 1背景
学习使用angular material时，看到了将一个模板（也就是html）中定义的变量赋值给了属性，实现了点击按钮触发mat-menu菜单。
```html
<button class="mat-button" [matMenuTriggerFor]="menuButtons">菜单</button>
<mat-menu #menuButtons="matMenu">
  <button class="mat-menu-item">按钮1</button>
  <button class="mat-menu-item">按钮2</button>
  <button class="mat-menu-item">按钮2</button>
</mat-menu>
```
模板引用变量就是，给一个标签起一个命名，比如#Var，然后可以在当前模板中引用Var的值；也可以将Var的值传入到ts中进行，后续的处理，背景中提到的就属于这种类型。根据这两种方法，分别举例模板引用变量的使用。
# 2模板内直接使用模板引用变量
模板引用变量使用sharp号(#)来声明引用变量。
模板引用变量通常用来引用模板中的某个DOM元素，比如，引用Angular组件或指令或 Web Component。
我们可以在当前模板的任何地方使用模板引用变量，示例：
```html
  <div class="panel panel-primary">
    <div class="panel-heading">
      <div class="panel-title">模板引用变量1</div>
    </div>
    <div class="panel-body">
      <input type="text" name="phone" placeholder="请输入手机号" class="form-control" 
      #phone (blur)="showOne(phone.value);"><br>
      <span class="lable lable-primary">{{phone.value}}</span>
    </div>
  </div>
```
```js
//ts文件，组件
  showOne(str: string) {
    console.info(str);
  }
```
# 2指令、组件等
典型的，就是背景中的例子：
```html
<button class="mat-button" [matMenuTriggerFor]="menuButtons">菜单</button>
<mat-menu #menuButtons="matMenu">
  <button class="mat-menu-item">按钮1</button>
  <button class="mat-menu-item">按钮2</button>
  <button class="mat-menu-item">按钮2</button>
</mat-menu>
```
在模板中声明了matMenu类型的组件，组件名称为menuButtons，毕竟这个变量传递给了button的属性，作为属性绑定用到的值，从而实现按键触发弹出。其实，不需要申明matMenu这个类型，也能实现绑定，属性绑定本质上只需要知道绑定的组件名称是什么，其实不关心组件的具体类型。
再举个例子，声明了Ngform类型的模板变量/对象：
```html
  <div class="panel panel-warning">
    <div class="panel-heading">
      <div class="panel-title">模板引用变量2-表单</div>
    </div>
    <div class="panel-body">
      <form (ngSubmit)="onSubmit(stuForm)" #stuForm="ngForm">
        <div class="form-group">
          <label for="name">姓名：</label>
          <input type="text" name="name" required [(ngModel)]="stu.name" class="form-control">
        </div>
 
        <div class="form-group">
          <label for="age">年龄：</label>
          <input type="number" name="age" required [(ngModel)]="stu.age" class="form-control">
        </div>
     
        <button *ngIf="!issubmit" class="btn btn-success" type="submit" [disabled]="!stuForm.form.valid">确定提交</button>
        <button *ngIf='issubmit' type="submit" disabled class="btn btn-success">正在提交...</button>
      </form>
 
      <div class="alert alert-info">
 
        表单对象:{{stu|json}}
      </div>
    </div>
  </div>
```
```js
//ts文件，组件
export class StudentComponent implements OnInit {
 
  constructor() { }
  ngOnInit() {
  }
 
  stu = {};//空对象
  public issubmit: boolean = false;
 
  showOne(str: string) {
    console.info(str);
  }
 
  onSubmit(model: NgForm) {
    console.info(model);
 
    //因为只读不能设置操作按钮的disabled属性
    //model.invalid=false;
    this.issubmit = true;
    setTimeout(() => {
      this.issubmit = false;
    }, 1000);
  }
}
```

