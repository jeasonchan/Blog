@[TOC]
# 1 安装
前提是已经安装了nodejs和npm然后无其他要求。
为什么要安装nodejs和npm?理由如下：
1. angular本身时利用js或者ts实现框架，想在本地运行需要js的解释器，nodojs即为脱离了浏览器的js解释器
2. angular本质上就是js的第三方包，因此，还是可以 通过npm进行安装、管理的
3. angular本身就是前后端分离的实现，nodejs“服务器”直接接收浏览器的网络请求，再将网络请求经过处理再转发给后端的服务器，nodejs接收到后端返回的数据后，进行处理，并将网页进行变化再将网页以字符串的形式（html本质上就是字符串）发送给浏览器，浏览器进行渲染，因此，做好controller在前端，正因为nodejs具有一定的处理能力，也可以直接作为轻量级的服务器存在。
```bash
sudo npm  -g  install  @angular/cli
```
以管理员权限，全局安装，angular/cli 模块，因此，必然会在 /usr/lib 中写入一个执行命令名称，就是"ng"，和"npm"一样，是全局函数名称。这种方式，默认按住最新的版本。
由于Ubuntu的nodejs还是8.x版本的，暂不支持最新版angular，因此，安装特定版本的angular：
```bash
sudo npm install -g   @angular/cli@7 .1.4
```
安装结束之后，通过安装列表查看是否已经安装：
```bash
jeason@Y400:~/IdeaProjects/angularStudy$ cnpm list -g |grep angular
└─┬ @angular/cli@7.1.4
  ├─┬ @angular-devkit/architect@0.11.4
  │ ├── @angular-devkit/core@7.1.4 deduped
  ├─┬ @angular-devkit/core@7.1.4
  ├─┬ @angular-devkit/schematics@7.1.4
  │ ├── @angular-devkit/core@7.1.4 deduped
  ├─┬ @schematics/angular@7.1.4
  │ ├── @angular-devkit/core@7.1.4 deduped
  │ ├── @angular-devkit/schematics@7.1.4 deduped
  │ ├── @angular-devkit/core@7.1.4 deduped
  │ ├── @angular-devkit/schematics@7.1.4 deduped
```
可见，已经成功安装，系统中已经包含相关的包。
# 2 初始化/新建项目
随便找个文件夹，然后在这个文件夹下面新建一个angular工程，通过：
```bash
ng  new
```
然后会让你输入工程名称，比如 "demo1"，然后会自动建立一个叫"demo1"的文件夹，并在这个文件夹内部生成一些文件，比如：package.json，这个package.json是nodej项目的典型标志，里面记录了nodejs项目的名称、依赖的包等等。一般情况下，由于网络原因，项目的初始化不会成功，这时候，cd到package.json所在文件夹，运行"npm install"，这时候会自动安装确实的依赖包，安装完成后，项目初始化就完成了。
可以运行相应的脚本命令，从而运行相应shell命令，所有的nodejs项目都是一样的，和npm中极为相似。
# 3 运行项目
两种方式，cd到package.json所在的文件夹，使用：
```bash
npm start  # 使用 package.json中定义的scripts进行项目启动，本质还是调用ng 自己的脚本
```
或者
```bash
ng server #使用angular.json中定义的脚本
```
所以，自定义启动参数时，可以在这两处进行相应修改，定制启动行为，比如host什么的
