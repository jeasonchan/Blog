# 1背景
写公司项目时，看到有人在angular中使用了async、await还有Promise，隐约感觉和Future类似；显然易见，是和js中的异步机制有关，来学习一下。

文章参考了<https://www.jianshu.com/p/1e75bd387aa0>。

## 1.1为什么需要async和await关键字
在async/await之前，我们有三种方式写异步代码：
1. 嵌套回调
2. 以Promise为主的链式回调
3. 使用Generators

但是，这三种写起来都不够优雅，ES2015之后做了优化改进，async/await应运而生，async/await相比较Promise对象then函数的嵌套，与 Generator 执行的繁琐（需要借助co才能自动执行，否则得手动调用next() ）， Async/Await 可以让你轻松写出同步风格的代码同时又拥有异步机制，更加简洁，逻辑更加清晰。

# 2 async语法
作用是：
1. 用于在定义函数时申明该函数是异步函数，返回值强制为Promise对象。因此，如果函数显式声明返回值为其他类型，编辑器静态检查都不通过，直接报红；所以，如果想声明类型，则必须声明为Promise<[具体的class]>；但是！自己在函数内部return的值不需要特地放到new出来的Promise中，会自动进行包装。比如：
```js
export async function jie_cheng(endNumber: number): Promise<number> {
    let result: number = 1;
    let flag: number = 1;
    while (flag <= endNumber) {
        result = result * flag++;//先使用，再自增
    }
    return result;//此处不需要返回Promise<number>类型
}
```
2. 只有async函数内部的Promise异步操作全执行完，才会执行then方法指定的回调函数
## 2.1代码实践
```js
console.log("line 3");
let result: Promise<number> = Util.jie_cheng(20);
result.then((resolve) => {
    console.log(resolve);
});
console.log("line 8");
```
打印结果为：
```
line 3
line 8
2432902008176640000
```
# 3 await语法
作用是：
1. 只能在async定义的函数内部使用
2. async 定义的方法本身就是异步，如果其内部再调用异步方法，就显然没有必要了。因此，为了使异步方法同步执行，就产生了await关键字，await的作用是就是等待异步方法执行完成，并取出异步方法返回的Promise对象中包装的值/对象。
## 3.1代码实践
如果没有await方法，现在有如下的方法:
```js
export async function Http_get(url: string) {
    console.log("line 13");
    let result;
    Request.get(url, (error, response, body) => {
        result = {
            "error": error,
            "response": response,
            "body": body
        }
        console.log(result);
    });
    
    //由于前面的get方法为异步方法，直行到这一步时，result仍是undefined
    console.log(result);
    
    //由于前面的get方法为异步方法，执行到这一步时，result仍是undefined
    //直接返回了undefined
    return result;
}
```
调用该方法：
```js
let url: string = "https://www.baidu.com/";
Util.Http_get(url).then((resolve) => {
    //打印undefined，并不会等待get操作执行完
    //因为，直接上面的方法return了，并且不是一个假的Promise对象，
    //假在Promise里装的是一个同步结果，undefined
    console.log("line 14",resolve);
});
```
如果可以使用await，则变得十分简单：
```js
export async function Http_get(url: string) {
    //实际上这一步是无效的！！！！！
    //因为，Request.get的返回值不是Promise类型，Request.get的返回值要进行类似toPromise的操作，await才有效
    return await Request.get(url)
```
await使基于Promise异步机制的异步调用，转换为同步函数，变得十分简单。
# 4 async和await联合使用





