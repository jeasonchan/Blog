```js
//if else 结构和java cpp大同小异，跳过

//switch() case constant-expression
//expression 是一个常量表达式，必须是一个整型或枚举类型

enum ColorEnum {
    Blue = "blue",
    Red = "red",
    Yellow = "yellow"
}

console.log(ColorEnum.Blue)

function checkColor(inputColor: string): string {
    inputColor = inputColor.toLowerCase();
    switch (inputColor) {
        case ColorEnum.Blue:
            return inputColor;
        case ColorEnum.Red:
            return inputColor;
        case ColorEnum.Yellow:
            return inputColor;
        default:
            return "unKnown color!";
    }

}

console.log(`checkColor("RED")is ${checkColor("RED")}`);
console.log(`checkColor("green")is ${checkColor("green")}`);

/*
运行结果是：
PS D:\Users\XXXX\Desktop\ts_project> tsc .\typescript条件语句.ts;node .\typescript条件语句.js
blue
checkColor("RED")is red
checkColor("green")is unKnown color!
*/
```
