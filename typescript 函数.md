```js
//typescript的函数定义和java十分类似，有点不同的地方在于：
//1、可以定义匿名函数
//2、可以以变量名的方式传递函数的引用地址
//3、直接使用lambda表达式定义匿名函数
//4、可能还有其他不同点，待发掘


//最正常的方式定义函数
function myfuntion(): void {
    console.log("最正统的函数定义方式");
}
myfuntion();//进行函数调用

//传递函数的引用地址，来“定义”新的函数
var myfuntion2 = myfuntion;
myfuntion2();

//定义匿名函数
var myfuntion3 = function () {
    console.log("使用function(){......}的方式定义匿名函数");
}
myfuntion3();

//定义匿名函数并使用，前端里面经常这么用！！！
(function () {
    console.log("定义匿名函数并立即使用！一次性的函数常这么定义使用！")
})();

//使用lambda表达式定义函数
var myfuntion4 = () => {
    console.log("使用lambda表达式定义函数并传递的地址给某一个变量")
};
myfuntion4();

//入参可以像Java一样不定长，同样的必必须放在入参的最后一个
//并且，在函数内部使用时作为的数组来使用
function myfuntion5(firstName: string, ...restName: string[]) {
    console.log(`first name is ${firstName} and restName is ${restName.join(" ")}`);
}
myfuntion5("chan");
myfuntion5("chan", "123", "6666");

//入参可以包含默认参数
//默认参数也是可以不输入的，是一种特殊的可选参数的处理逻辑
function myfuntion6(firstName: string, secondName: string = "default string") {
    console.log(`first name is ${firstName} and second name is ${secondName} by default`);
}
myfuntion6("chan");

//入参包含可选参数
function myfuntion7(firstName?: string) {
    if (firstName) {
        console.log(`input arg is ${firstName}`);
    } else {
        console.log("no input arg");
    }
}
myfuntion7();
myfuntion7("chan");

//
var test_value;
//变量已声明且时any类型，未赋值，为undefined，也可以作为condition判断条件
if (test_value) {
    console.log(true);
} else {
    console.log(false);
}

test_value = ""
//赋值之后，字符串虽然已经有值，但是，值并不是为true，所以还是打印false
if (test_value) {
    console.log(true);
} else {
    console.log(false);
}

/*
编译输出：
PS C:\CRroot\documents\codeproject\tampermonkey-scriptps> tsc .\typescript函数.ts;node
.\typescript函数.js
最正统的函数定义方式
最正统的函数定义方式
使用function(){......}的方式定义匿名函数
定义匿名函数并立即使用！一次性的函数常这么定义使用！
使用lambda表达式定义函数并传递的地址给某一个变量
first name is chan and restName is
first name is chan and restName is 123 6666
first name is chan and second name is default string by default
no input arg
input arg is chan
false
false
*/
```
