@[TOC]
# 1 背景
对input采用ngModule双向数据绑定时，浏览器控制台爆出标题的错误，去appMdoule.ts查看，发现已经"import { NgModule } from '@angular/core';"，但是仍然报错，很是费劲，毕竟自己使用的就是：
```html
<label>数字1</label><input type="number" [(ngModel)]="number1"><br>
```
很是费解，经过百度之后发现有很多相同的问题，解决方案是：
1. app.module.ts中 import { FormsModule, ReactiveFormsModule } from '@angular/forms';
2. app.module.ts中，进行模块使用导入，加上"FormsModule"即可
```js
imports: [
    BrowserModule,
    FormsModule
  ]
```
# 2 相应知识总结
angualr中，
1. 同一个module下的components之前互相调用，直接在ts中import相应的类即可，在ts中已经exports的前提下，使用"import { Util } from './../util';"进行导入
2. 同样的，就算不是同一个模块，只要是导入component，都可以直接import，比如"import { Component, OnInit } from '@angular/core';"
3. 要想导入模块，则必须在模块的ts文件中进行导入，是不能在组件（component）ts文件中导入 module的，因此，ngModel依赖的FormsModule必须在app.module.ts中进行import，但是！！！为了让导入的module被该module下component使用，同时还需要在imports列表中加入
4. import { NgModule } from '@angular/core';中的ngModel是“装饰器”，@NgModule()进行模块信息描述时进行了使用。
