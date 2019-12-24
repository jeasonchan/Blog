```js
//创建数组
var array01: number[] = [1, 2, 3, 4, 5];
var array02: Array<any> = [1, 2, 2, 3, "string"];
var array03: Array<number> = new Array(1, 2, 3, 4);


//感觉元组就是一种r特殊的数组
var variable: [string, string] = ["1", ""];
variable.push("true");
console.log(variable);
variable.shift();
console.log(variable);


//试试数组的一些方法
for (var each in array01) {
    console.log(each);
}

console.log(array01.concat(array02));

function checkBiggerThan4(element: number, index: any, array: any) {
    console.log(`now is checking ${element} and result is ${element < 4}`);
    return (element as number) < 4;
}
//返回true，相当于continue；返回false，相当于break；
array01.every(checkBiggerThan4);

array01.forEach((each, index, array) => console.log(`第${index}个元素是${each}`));

var result: string = "**";
console.log(array02.join(result));

console.log(array01.map(
    (each) => {
        //一定要有返回值！返回值会作为map出来的新值放进一个新数组
        return each * 2 + 1;
    }
)
);


var ouuputResult: any =
    array01.reduce((previous, current, currentIndex, outputArray) => {
        console.log(`${previous}+${current}=${previous + current}`);
        return previous + current;
    }
    );
console.log(ouuputResult);

/*
PS C:\CRroot\documents\codeproject\tampermonkey-scriptps> tsc .\typescript_数组.ts;node .\typescript_数组.js
[ '1', '', 'true' ]
[ '', 'true' ]
0
1
2
3
4
[ 1, 2, 3, 4, 5, 1, 2, 2, 3, 'string' ]
now is checking 1 and result is true
now is checking 2 and result is true
now is checking 3 and result is true
now is checking 4 and result is false
第0个元素是1
第1个元素是2
第2个元素是3
第3个元素是4
第4个元素是5
1**2**2**3**string
[ 3, 5, 7, 9, 11 ]
1+2=3
3+3=6
6+4=10
10+5=15
15
*/
```
