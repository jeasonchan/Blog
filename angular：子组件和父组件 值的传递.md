
# 0 结论先行
**个人已知的方法，但是，还不清楚最佳实践是哪种：**

父组件向子组件传递值的方法：
1、直接在使用的父组件的模板里，使用值的单项绑定，比如，\<child  [data]="father_value"\>\<child\\\>
2、通过viewchild，完全在ts中子组件的值进行写入。有个缺点，如果ts代码写的烂，接手的人可能看不懂……

子组件向父组件传递值的方法：
1、通过事件，传递值到父组件的函数里，有个前提是，要有触发事件
2、在模板里，给子组件命名，在通过"名称.变量名"的方式将值传递给函数
3、在ts里，viewchild，父组件的函数直接通过"this.子组件名称.变量名"的方式，实现对子组件值的访问

# 1 父组件向子组件传递值
父组件：

father.template.html
```html
<h1>父组件</h1>
<cmt-child [data]='data'></cmt-child>
```
father.component.ts
```js
import { Component, OnInit } from '@angular/core';

@Component({
    selector: 'cmt-father',
    templateUrl: './father.template.html'
})
export class FatherComponent implements OnInit {
    data: any = '我是传往子组件的值'
    ngOnInit() {
    }

    ngOnChanges() {
    }

}
```

子组件：（使用@Input修饰器去接收）
childcomponent.ts
```js
import { Component, OnInit, Input } from '@angular/core';

@Component({
    selector: 'cmt-child',
    templateUrl: './child.template.html'
})
export class ChildComponent implements OnInit {
    @Input() data: any;//接收父组件的值
    ngOnInit() {
        console.log(this.data)
    }

    ngOnChanges() {
        console.log(this.data)
    }

}
```
这样就把值从父组件传到了子组件！

# 2 子组件向父组件传值
# 2.1 通过事件的方式传递
（子组件通过方法借助修饰器@output传值给父组件）

子组件
childcomponent.ts
```js
import { Component, OnInit, Input, Output, EventEmitter } from '@angular/core';

@Component({
    selector: 'cmt-child',
    templateUrl: './child.template.html'
})
export class ChildComponent implements OnInit {
    @Output('checked') checkedBack = new EventEmitter<any>();
    id:any ="我是传给父组件的值"
    ngOnInit() {
    }

    ngOnChanges() {
    }
    checkedCallback() {
        this.checkedBack.emit(this.id);
    }
}

```

child.template.html.html
```html
<h1>子组件</h1>
<button (click)='checkedCallback()'>点击传值给父组件</button>
```

father.template.html
```html
<h1>父组件</h1>
<cmt-child (checked)="checkedBack($event)"></cmt-child>
```
father.component.ts

```js

import { Component, OnInit } from '@angular/core';

@Component({
    selector: 'cmt-father',
    templateUrl: './father.template.html'
})
export class FatherComponent implements OnInit {
    ngOnInit() {
    }

    ngOnChanges() {
    }
    
    checkedBack(event) {
        console.log(event)
    }
}

```

这样子组件通过点击就把值传递给了父组件！
# 2.2直接通过组件对外暴露的属性
子组件
childcomponent.ts
```js
import { Component, OnInit, Input, Output, EventEmitter } from '@angular/core';

@Component({
    selector: 'cmt-child',
    templateUrl: './child.template.html'
})
export class ChildComponent implements OnInit {
    @Output('checked') checkedBack = new EventEmitter<any>();
    id:any ="我是传给父组件的值"
    ngOnInit() {
    }

    ngOnChanges() {
    }
    checkedCallback() {
        this.checkedBack.emit(this.id);
    }
}

```
father.template.html
```html
<h1>父组件</h1>
<cmt-child #childName (checked)="checkedBack(childName.id)"></cmt-child>
```
father.component.ts

```js

import { Component, OnInit } from '@angular/core';

@Component({
    selector: 'cmt-father',
    templateUrl: './father.template.html'
})
export class FatherComponent implements OnInit {
    ngOnInit() {
    }

    ngOnChanges() {
    }
    
    checkedBack(input:any) {
        console.log(input)
    }
}

```
