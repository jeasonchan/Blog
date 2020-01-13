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
**如果**可以使用await，则变得十分简单：
```js
export async function Http_get(url: string) {
    //实际上这一步是无效的！！！！！
    //因为，Request.get的返回值不是Promise类型，Request.get的返回值要进行类似toPromise的操作，await才有效
    return await Request.get(url)
```
await使基于Promise异步机制的异步调用，转换为同步函数，变得十分简单。
# 4 async和await联合使用
## 4.1 异常捕捉
在async函数里，await会自动解析返回的Promise对象，是resolve则返回，是reject的则自动throw；所以最好把await放入try{}catch{}中，catch能够捕捉到Promise对象rejected的数据或者抛出的异常，比如：
```js
function timeout(ms) {

  return new Promise((resolve, reject) => {

    setTimeout(() => {reject('error')}, ms);  //reject模拟出错，返回error

  });

}

async function asyncPrint(ms) {

  try {

    console.log('start');

    //这里返回了错误,await相当于自动解析了Promise返回值
    //是resolve则返回，是reject的则自动throw
    await timeout(delayTme_ms);

    console.log('end');  //所以这句代码不会被执行了

  } catch(err) {

     console.log(err); //这里捕捉到错误error

  }

}

//==============开始调用==================
asyncPrint(1000);
```
捕捉异常的地点还有多种，比如：

在最终调用异步方法时捕捉：
```js
//改写原先的异步函数
export async function asyncPrint(delayTme_ms: number) {
    console.log("start");
    await timeout(delayTme_ms);
    console.log("end");
}

//调用
Util.asyncPrint(12).catch(reason => {
    console.log(reason);
}).then(value => {
    console.log(value);
});

```
在内部的等待异步方法完成时调用：
```js
export async function asyncPrint(delayTme_ms: number) {
    console.log("start");

    //不让await解析异常，自己手动提前将异常捕捉，
    //await对Promise对象进行解析时，已经异常了，只有value了
    await timeout(delayTme_ms).catch(error => {
        console.error(error);
    });

    console.log("end");
}
```
**异常捕捉总结**

await会对的Promise对象进行解析，是resolve则返回，是reject的则自动throw；捕捉方式就分为有try catch和Promise.catch两种，Promise.catch是自己主动catch掉异常，不让await捕捉/自动抛出，以达到不阻断后续语句运行的目的。
# 5  Promise.all“并发”执行
await 异步操做变为同步的操作，有时候也会有缺点，比如：
```js
/**
 *  睡眠时间大于2000，则先睡眠到相应时间，然后抛出异常；
 *  小于等于2000，则睡眠到相应的时间，正常退出。
 * @param delayTime 单位 毫秒
 */
export async function sleep(delayTime: number) {
    return new Promise((resolve, reject) => {
        setTimeout(() => {
            if (delayTime > 2000) {
                reject(`sleep ${delayTime} ms, and ${delayTime}>2000, too long!`);
            }

            //正常情况下的，resolve也不能放值，因为是void类型
            console.log(`sleep ${delayTime} ms`);
            resolve(`sleep ${delayTime} ms`);
        }, delayTime);
    });
}

export async function sleep_500_and_1000_method1() {
    let endTime: number;
    let starTime: number = Date.now();
    console.log(`start time:${starTime}`);

    //以下两步各个都是同步执行时，两者是串行的
    await sleep(500).catch(error => console.log(error));
    await sleep(1000).catch(error => console.log(error));

    endTime = Date.now();
    console.log(`end time:${endTime} and cost ${endTime - starTime}`);
}

export async function sleep_500_and_1000_method2() {
    let endTime: number;
    let starTime: number = Date.now();
    console.log(`start time:${starTime}`);

    //以下两步分别是同步执行时，但是，开始执行的时间相同
    await Promise.all([sleep(500), sleep(1000)]);

    endTime = Date.now();
    console.log(`end time:${endTime} and cost ${endTime - starTime}`);
}

sleep_500_and_1000_method1();
sleep_500_and_1000_method2();


```
输出：
```
start time:1578904081654
start time:1578904081668
sleep 500 ms
sleep 500 ms
sleep 1000 ms
end time:1578904082662 and cost 994
sleep 1000 ms
end time:1578904083172 and cost 1518
```
查看输出发现，使用await配合Promise.all()同时处理/等待多个Promise对象时，能解决更多时间。

但是，运行过程中发现：new Promise对象时，如果回调函数内部没有resolve操作，虽然得到的是个一个Promise对象，但是并不能异步……并不知道为什么……



