@[TOC]
# 1 什么是ts的断言
类型断言（Type Assertion）可以手动指定一个值的类型，有点类似于java中的转型，都是一种临时性的类型声明，并不会对原本的引用的类型发生更改。
语法如下：
```js
var a: any = "123";
console.log((<string>a).length)
console.log((a as string).length)
```
两种显式地进行断言语法，虽然，实际过程中我不进行任何断言，vscode也自动识别出a有length这个属性……但是，显式地进行断言还是更加规范合必要的。

# 2 为什么要进行断言？
因为：ts和java不同，ts支持联合类型，即一个变量的类型可是多种，甚至是互不兼容的多种；同时，ts又是静态类型和强类型的，有些属性和方法只能对某个类型使用，因此，为了明确操作的联合类型变量的类型，此时就要使用类型断言。
类似的，在java中，父类向子类断言/转型，只有原本就是该子类的情况下才能进行转型，从而能顺利使用子类的方法、属性；而子类没必要向父类转型，因为方法默认时继承的，子类也能使用父类的方法、属性。
举例说明将一个联合类型的变量指定为一个更加具体的类型。
当 TypeScript 不确定一个联合类型的变量到底是哪个类型的时候，我们只能访问此联合类型的所有类型里共有的属性或方法：
```js
    function getLength(something: string | number): number {
        return something.length;
    }
     
    // index.ts(2,22): error TS2339: Property 'length' does not exist on type 'string | number'.
    //   Property 'length' does not exist on type 'number'.
```

而有时候，我们确实需要在还不确定类型的时候就访问其中一个类型的属性或方法，比如：
```js
    function getLength(something: string | number): number {
        if (something.length) {
            return something.length;
        } else {
            return something.toString().length;
        }
    }
     
    // index.ts(2,19): error TS2339: Property 'length' does not exist on type 'string | number'.
    //   Property 'length' does not exist on type 'number'.
    // index.ts(3,26): error TS2339: Property 'length' does not exist on type 'string | number'.
    //   Property 'length' does not exist on type 'number'.
```

上例中，获取 something.length 的时候会报错。
此时可以使用类型断言，将 something 断言成 string：
```js
    function getLength(something: string | number): number {
        if ((<string>something).length) {
            return (<string>something).length;
        } else {
            return something.toString().length;
        }
    }
```
    或
```js
      function getLength(something: string | number): number {
        if ((something as string).length) {
            return (something as string).length;
        } else {
            return something.toString().length;
        }
    }
```
类型断言的用法如上，在需要断言的变量前加上 <Type> 即可。类型断言不是类型转换，断言成一个联合类型中不存在的类型是不允许的（和java中不能强行转为不兼容的子类有点相似）：
```js
   function toBoolean(something: string | number): boolean {
        return <boolean>something;
    }  
    // index.ts(2,10): error TS2352: Type 'string | number' cannot be converted to type 'boolean'.
```
