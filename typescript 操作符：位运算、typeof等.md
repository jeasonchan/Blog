```js
//位运算，typeof 运算法
let byte = 5 << 1;
console.log(`byte value is ${byte} and type is ${typeof byte}`);

//和java、cpp中的一样，右边自增操作，是直接取原始值；
//取值的语句执行完之后，值才会发生变化
let number2: number = 1;
console.log("number2++的值是：" + number2++);
console.log("执行过number2++后，number的值是：" + number2);

//mod运算，就是取余数
console.log(`7%5的值是${7 % 5}`);
//除法运算，就是真实的除法运算
console.log(`1/3的值是${1 / 3}`);
//支持除数为0，结果为无穷大
console.log(`1/3的值是${1 / 0}`);

//三元表达式
function checkIfBggerThanZero(input: number): string {
    return input > 0 ? "大于0" : "小于0";
}
console.log(checkIfBggerThanZero(111));


/*
运行结果：
PS D:\Users\XXXXXXX\Desktop\ts_project> tsc .\typescript操作符.ts;node .\typescript操作符.js
byte value is 10 and type is number
number2++的值是：1
执行过number2++后，number的值是：2
7%5的值是2
1/3的值是0.3333333333333333
1/3的值是Infinity
大于0
*/
```
