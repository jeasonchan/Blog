@[TOC]
# 1 背景
学习angualr时，使用ngClass和ngStyle一脸懵逼，两者都是对标签的属性进行个性化自定义的，但是，在使用起来的过程中有时候会出现交替使用的情况，十分费解，特地学习并总结了一下style和class
# 2 联系和区别
先上结论：
## 2.1 联系
1. class和style都是对标签的属性进行个性化设置，常见的属性及使用见本文的第三节
2. class可以说是可复用的style的集合
## 2.2 区别
1. class一般是定义在css(层叠样式表)文件里的，同类型的标签可以对css文件中的class进行复用；而style一般都是
2. class除了可以当作style集合使用以外，还可以使用纯css实现动画效果
3. class定义时一般是针对某个标签类定义其对应的class，具有一定的针对行，比如```"div.myClass{……}"	```而style一般是现定义现用，且不会复用，定义的时候不会有针对性，直接```style="xxxxxx"````就完事了
4. style和class同时对标签的某些属性进行了个性化时，style中的属性设置优先使用。
举个例子说明style和class的区别和联系
class所在的层叠样式表文件，**style.css**
```css
/*
在样式表里面里面，针对div标签，定义一个的divClass样式类
*/
div.divClass{
border-top: red 1px solid;
}

/*
一个通用的样式
 */
.universalClass{
  border-top: red 1px solid;
}
```
原生html静态页面，**test.html**
```html
<html>
<!--导入样式表-->
<link rel="stylesheet" type="text/css" href="style.css" />
<!--分别使用class和style实现同div上的同一个样式-->
<div id="div1" style="border-top: red 1px solid;">
<div id="div2" class="divClass">
<span class="divClass">尝试对span加载div的样式表，并不起作用</span>
<hr>
<span class="universalClass">尝试对span加载通用样式表</span>
<div class="universalClass">尝试对div加载div的样式表</div>
</html>
```
最终，上边的两个div的样式是相同的，两种个性化的内容是相同的。
# 3 style常见属性及实践
```html
<strong>开始尝试一些常用的样式属性</strong>
<div style="width:200px;height:100px;background-color:green;">
  宽度width<br>
  高度height<br>
  背景色background-color
</div>
<hr>
<div style="border:black 1px solid;width:500px;height:200px;">父容器
  <div style="margin:5px 10px 20px 30px;width:100px; height:100px;background-color:red;">
  </div>
</div>
<span>
  margin:后面跟有四个距离分别为到父容器的上-右-下-左边的距离，可以看例子中的红色子div到父div的描边距离。<br>
  还可以分别设置这四个边的距离，用到的属性如下：<br>
  margin-left:到父容器左边框的距离。<br>
  margin-right:到父容器右边框的距离。<br>
  margin-top: 到父容器上边框的距离。<br>
  margin-bottom:到父容器下边框的距离。
</span>
```
效果如下：
![实际效果](https://img-blog.csdnimg.cn/20191026165203661.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2plYXNvbl9jaGFuX3pqdQ==,size_16,color_FFFFFF,t_70)
更多的常见属性和实践代码，可参考<https://blog.csdn.net/qq_38211852/article/details/80290516>。

# 4 总结
class可以说是可复用的style的集合（但功能不仅于此）；不指明对应组件/标签类型的样式类具有通用性，否则无法使用（改变标签对应的属性值），就算有对应的属性也无法使用。
