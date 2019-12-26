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

# 3jQuery的事件绑定
