@[TOC]
# js输出
js没有专门的输出打印函数，但是可以通过不同的方式来输出数据：
* 使用window.alert()弹出警告框
* 使用document.write()讲内容写到html文档中
* 使用innerHTML写入到HTML元素
* 使用console.log()写入到浏览器的控制台

# 代码实践及注释
```html
<!DOCTYPE html>

<html>
<script>
    document.write("<p>已加载页面！</p>")
</script>

<head>
    <p id="head_p">我是html的head！</p>
</head>

<body>
    <script>
        //使用windows.alert方法进行打印输出
        window.alert(5 + 6)  //并且发现，弹出警告窗时，页面还都没加载处理出来，点击确认后，才出现html的相关文字
        window.alert("第二个警告！") //第二个警告会在第二上一个弹出后，继续弹出，让用户进行确认

        document.getElementById("head_p").innerText = "此内容是js修改后显示的内容1！"
        //innerText是取到的这个element的属性，此语句的含义是，将该字符串复制给这个元素的innerText属性

        document.getElementById("head_p").innerHTML = "此内容是js修改后显示的内容2！"//与上同理

        document.write("现在的时间是：")  //被浏览器识别为text
        document.write("<div></div>")//这是用来控制换行的，浏览器识别为div
        document.write(Date()) //被浏览器识别为text
    </script>


    <!-- 在点击按钮之前，html文件已经加载完毕。这时，如果再执行document.write()操作，整个页面都会被覆盖！ -->
    <div><button onclick="docWrite()">点我</button> </div>
    <script>
        function docWrite() {
            document.write(Date())
        }
    </script>

    <script>
        a = 1
        b = 2
        c = a + b
        console.log(c)
    </script>

</body>

</html>
```

