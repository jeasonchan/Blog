@[TOC]
先吐槽一下，有些沙雕，上来介绍安装angualr material，也不让我们加版本号……目前material最新版是8.x版本，如果angualr cli安装的不是8.x版本，会不兼容……所以，**angular cli和material大版本一定要匹配！**
# 1安装material
0. 路径切换到项目根目录
1. ng --version 确认自己的angualr cli版本，比如我的是7.x
2. 安装必要三个依赖
```bash
# 一定要加上和自己大版本一直的版本号，我是是@7
npm install --save @angular/material@7 @angular/cdk@7
# 这个负责动画，不装这个，那些弹出的动画完全用不了，比如弹出菜单mat-menu
npm install --save @angular/animations@7
```
3. 根据提示，查漏补缺，把需要的依赖都补上，**务必明确大版本号**
# 2代码实践
以使用Mat-menu为例，简述使用过程：
1. app.module导入动画模块和菜单（Menu）模块
```js
...
import {MatMenuModule} from '@angular/material';
import {BrowserAnimationsModule} from '@angular/platform-browser/animations';
...
  imports: [
    ...
    MatMenuModule,
    BrowserAnimationsModule
  ],
```
2. 全局导入material的css文件
**style.css**
```css
/* You can add global styles to this file, and also import other style files */
@import "~@angular/material/prebuilt-themes/indigo-pink.css";
```
3. 模板中使用菜单标签
```html
<button class="mat-button" [matMenuTriggerFor]="menuButtons">菜单</button>
<mat-menu #menuButtons="matMenu">
  <button class="mat-menu-item">按钮1</button>
  <button class="mat-menu-item">按钮2</button>
  <button class="mat-menu-item">按钮2</button>
</mat-menu>
```
mat-menu本质上只包含弹出的动画和弹出的东西，不包括显示“菜单”二字的按钮，因此，“菜单”按钮需要绑定这个能弹出的mat-menu实例，来实现点击弹出菜单内容。

4. 不要从ts中取数据，这个方法没有采用ts中动作控制程序弹出，还可以通过ts中的trigger实现弹出效果

# 3 通过MatMenuTrigger实现手动触发
```html
<button class="mat-button" (click)="someMethod()">菜单</button>
<mat-menu #menuButtons="matMenu"><--此处需要修改！！！！-->
  <button class="mat-menu-item">按钮1</button>
  <button class="mat-menu-item">按钮2</button>
  <button class="mat-menu-item">按钮2</button>
</mat-menu>
```
```js
class MyComponent {
  @ViewChild(MatMenuTrigger) trigger: MatMenuTrigger;
  someMethod() {
    this.trigger.openMenu();
  }
}
```
