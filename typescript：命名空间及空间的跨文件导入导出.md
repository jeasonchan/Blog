**namespace_exercise.ts**
```js
import { MyNameSpace_2 } from "./namespace_exercise_2";

//没用export 修饰的类、变量、函数等，导入命名空间后，也无法使用这些变量
//用了export 修饰的，只要导入了命名空间，
//就可以通过  namespace.var_name 的方式访问空间内的
namespace MyNameSpace {
    var myName: string = "jeason chan";
    export var yourName: string = "SB";
}

//在同一个文件中使用同一个文件中的命名空间中的对象
console.log(MyNameSpace.yourName);
//console.log(MyNameSpace.myName); 无法访问 myName，因为没导出


//向命名空间中添加变量
namespace MyNameSpace {
    export var newArg: string = "aaaaa";
}
//访问添加的变量
console.log(MyNameSpace.newArg);
//访问原先的变量
console.log(MyNameSpace.yourName);

//别的文件中，不先导入，相同命名空间内的对象也无法访问，
//因为，跨了文件之后，只有导入了能算在同一个命名空间
//console.log(MyNameSpace.ArgInOtherFile)  报错，不导入无法使用

/// <reference path = "./namespace_exercise_2.ts" />
MyNameSpace_2.printHello();
// console.log(MyNameSpace.ArgInOtherFile);  无法识别，因为并没有导出来

//命名空间支持嵌套

```
**namespace_exercise_2.ts**
```js
//不期望该命名空间导出，不加export
namespace MyNameSpace {
    export var ArgInOtherFile: string = "hahah";

}

//期望该命名空间导出，加export
export namespace MyNameSpace_2 {
    export function printHello(): void {
        console.log("Hello world!")
    }
}

//一定要严格遵守！！！
//希望在别的文件中使用的就用export！！！
//不希望的就不加！！！
export var hahahah: string = "aaaa";
```

