# 1背景
写油猴脚本时，不会jquery十分痛苦，暂且入一下jQuery的门，了解一下jQuery的基本语法、选择器和元素事件。

jQuery 语法是通过选取 HTML 元素，并对选取的元素执行某些操作。

基础语法： $(selector).action()
- 美元符号定义了 jQuery
- 选择符（selector）"查询"和"查找" HTML 元素
- jQuery 的 action() 执行对元素的操作
实例:
- $(this).hide() - 隐藏当前元素
- $("p").hide() - 隐藏**所有** \<p\> 元素
- $("p.test").hide() - 隐藏所有 class="test" 的 \<p\> 元素
- $("#test").hide() - 隐藏所有 id="test" 的元素

所有 jQuery 函数位于一个 document ready 函数中：
```js
$(document).ready(function(){
 
   // 开始写 jQuery 代码...
 
});
```
这是为了防止文档在完全加载（就绪）之前运行 jQuery 代码，即在 DOM 加载完成后才可以对 DOM 进行操作。如果在文档没有完全加载之前就运行函数，操作可能失败。下面是两个具体的例子：
- 试图隐藏一个不存在的元素
- 获得未完全加载的图像的大小
文档就需事件有简写方法，效果完全相同，都是DOM加载完再进行操作:
```js
$(function(){
 
   // 开始写 jQuery 代码...
 
});
```
# 2jQuery的选择器用法
jQuery 选择器允许您对 HTML 元素组或单个元素进行操作。jQuery 选择器基于元素的 id、类、类型、属性、属性值等"查找"（或选择）HTML 元素。 它基于已经存在的 CSS 选择器，除此之外，它还有一些自定义的选择器。

jQuery 中所有选择器都以美元符号开头：$()。

**元素选择器**

jQuery 元素选择器基于元素名选取元素。在页面中选取所有 <p> 元素:
```js
$(document).ready(function(){
  $("button").click(function(){
    $("p").hide();
  });
});
```

**#id 选择器**

jQuery #id 选择器通过 HTML 元素的 id 属性选取指定的元素。页面中元素的 id 应该是唯一的，所以您要在页面中选取唯一的元素需要通过 #id 选择器。通过 id 选取元素语法如下：
当用户点击按钮后，有 id="test" 属性的元素将被隐藏：
```js
$(document).ready(function(){
  $("button").click(function(){
    $("#test").hide();
  });
});
```

**.class 选择器**

jQuery 类选择器可以通过指定的 class 查找元素。
```js
//用户点击按钮后所有带有 class="test" 属性的元素都隐藏：
$(document).ready(function(){
  $("button").click(function(){
    $(".test").hide();
  });
});
```

**属性选择器**

```js
$("[href]")	//选取带有 href 属性的元素
$("a[target='_blank']")	//选取所有 target 属性值等于 "_blank" 的 <a> 元素
$("a[target!='_blank']")	选取所有 target 属性值不等于 "_blank" 的 <a> 元素	在线实例
```

更多实例:
```js
$("*")	//选取所有元素
$(this)	//选取当前 HTML 元素
$("p.intro")	//选取 class 为 intro 的 <p> 元素
$("p:first")	//选取第一个 <p> 元素
$("ul li:first")	//选取第一个 <ul> 元素的第一个 <li> 元素
$("ul li:first-child")	//选取每个 <ul> 元素的第一个 <li> 元素
$(":button")	//选取所有 type="button" 的 <input> 元素 和 <button> 元素
$("tr:even")	//选取偶数位置的 <tr> 元素
$("tr:odd")	//选取奇数位置的 <tr> 元素
```
如果网站包含许多页面，并且我们希望jQuery 函数易于维护，那么最好把 jQuery 函数放到独立的 .js 文件中。偷懒的时候，可以将函数直接添加到 <head> 部分中。不过，把它们放到一个单独的文件中会更好，就像通过 src 属性来引用文件：
```html
 <head>
<script src="http://cdn.static.runoob.com/libs/jquery/1.10.2/jquery.min.js">
</script>
<script src="my_jquery_functions.js"></script>
</head>
```
  
# 3jQuery的事件绑定
在事件中经常使用术语"触发"（或"激发"）例如： "当您按下按键时触发 keypress 事件"。

常见 DOM 事件：

|鼠标事件|	键盘事件	|表单事件|	文档/窗口事件|
|---|---|---|---|
| click |	keypress| submit | load |
|dblclick |	keydown	|change	|resize |
| mouseenter	| keyup	| focus	| scroll |
| mouseleave | blur |	unload |
| hover |

**jQuery 事件方法语法**

在 jQuery 中，大多数 DOM 事件都有一个等效的 jQuery 方法。页面中指定一个点击事件：
```js
$("p").click(function(){
    // 动作触发后执行的代码!!
});
```

**常用的 jQuery 事件方法举例**

**dblclick()** 当双击元素时，会发生 dblclick 事件。dblclick() 方法触发 dblclick 事件，或规定当发生 dblclick 事件时运行的函数：
```js
$("p").dblclick(function(){
  $(this).hide();
});
//所有的p标签都绑定了该属性，但是，this 对象限定了，只有当前dbclick的对象会相应dbclick操作
```

**mouseenter()** 当鼠标指针穿过元素时，会发生 mouseenter 事件。mouseenter() 方法触发 mouseenter 事件，或规定当发生 mouseenter 事件时运行的函数：
```js
$("#p1").mouseenter(function(){
    alert('您的鼠标移到了 id="p1" 的元素上!');
});
```
