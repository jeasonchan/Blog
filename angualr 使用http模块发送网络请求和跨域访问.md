@[TOC]
# 1代码实践
## 1.1 引入模块HttpClientModule 
app.module.ts里面引入HttpClientModule，分为两步：
* import { HttpClientModule } from '@angular/common/http';
* NgModule的元数据对象的，imposts属性中，追加HttpClientModule 
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191014141344448.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2plYXNvbl9jaGFuX3pqdQ==,size_16,color_FFFFFF,t_70)
## 1.2 使用模块中的HttpClient类
只需要一步：import { HttpClient} from '@angular/common/http';
然后就可以直接用了，http这个类实例是类的私有属性，其实可以作为私有属性直接写的，但是，实例化时new的时候不太好new……所以偷懒，用作为构造函数的入参这种方法去初始化私有属性。
调用httpclient的方法和异步处理subscribe实现异步加载。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191014143247427.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2plYXNvbl9jaGFuX3pqdQ==,size_16,color_FFFFFF,t_70)**注！！如果不在模块中先导入模块，而是在组件中直接使用这个类，静态检查能通过，但编译运行时无法识别出这类。**

# 2 跨域访问
本地运行时，当url设置为百度的一个get请求时，由于get的返回值中包含js脚本，因为就会涉及跨域访问问题，导致无法查看get请求的返回最结果。解决方法：
1. 项目的根目录proxy.config.json文件，内容为：
```json
{
	"/":{
		"target":"http://网址:端口" //这里是你的接口所在的位置
	}
}
```
2. 修改package.json文件中的scripts选项中的starts，
```json
"start": "ng serve --proxy-config proxy.config.json"
```
3. npm start 重新运行项目
4. 原来调用接口的代码不需要任何修改，比如：
```js
this.http.post(url,val).map(res => res.json()).subscribe(data=>{
	if (data) {
		this.title=data.msg;
	}
})
```
