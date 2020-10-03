# 1 背景

CPP中动态建立二维数组（1）  https://blog.csdn.net/yuqinjh/article/details/79095787

CPP中动态建立二维数组（2）   https://www.cnblogs.com/aminxu/p/4685962.html


# 2 笔记
在C++中，数组可以被视为一种类型。但是，不存在"二维数组"这种类型。二维数组本身会被解释成一个一维数组：这个数组的元素类型为另一种一维数组。比如int[2][3]这个二维数组，它会被编译器视作一个元素类型为"int[3]"的一维数组。并且，比如，"int[3]"和"int[4]"会被当成不同的数据类型。

假设a, b为两个int型变量，如果我们希望这样生成一个二维数组：new int[a][b]，是不会得到编译器允许的。数组的大小必须要能在编译器就确定下来，由于没有指定这个数组的元素类型，也就是b的大小未知，编译器无法就确定"int[b]"到底是一个什么类型。所以，要用new创建一个二维数组，这其中有讲究。

## 2.1 使用常量创建二维数组
接上面的问题，如果b是const int b=123;这样的常量，那么创建数组的语句就变成了new int[a][123]，其实质与new int[a]并没有多大区别，只不过类型是int[123]这样的类型。比如：

```cpp
void createFunction1(unsigned int n) {
    unsint i, j;
    const int b = 123;

    //elements in array is int[123] type
    //st like int[b] *array2D=new (int[123])[n]
    int (*array2D)[b] = new int[n][b];
    for (int height = 0; height < n; ++height) {
        for (int width = 0; width < b; ++width) {
            std::cout << array2D[height][width] << " ";
        }
    }

    delete[] array2D;
}

```
用这个方法来创建二维数组，比较直观、易用，但它最大的限制在于：你必须在编译时确定b的大小。



## 2.2 使用指针间接引用
首先创建若干个大小一致的动态数组，然后将这些数组的首地址(转化为指针)按顺序存储到一个动态数组中，就相当于模拟了一个二维动态数组。

```cpp
void createFunction2(unsigned int height, unsigned int width) {
    //element in array is int* type
    int **array2D = new int *[height];

    //create each row
    for (int i = 0; i < width; ++i) {
        *array2D = new int[width];
    }

    //destroy array
    for (int i = 0; i < height; ++i) {
        delete[] array2D[i];
    }
    delete[] array2D;

}
```

这个方法实现了两个维度的动态创建，访问也比较方便。但是有一个缺点：**由于低一级的数组是分开创建的，所以整个二维数组的内存不连续**——类似‘array2D[i * width + j]’这样的访问就不要使用了，容易造成访问越界。


## 2.3 使用vector