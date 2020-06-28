@[TOC]
# 1 形式
```java
label1:
{
	//statements
}
```
使用一个标签来标记一个代码块，使用break label1时，直接退出整个代码块，执行代码块下一行的语句；使用continue label1时，从代码块的第一行开始重新执行。因为这样方便跳转的特性，常常用于跳出深度的循环。
# 2 循环标签
```java
        label1:
        {
            while (true) {
                System.out.println("statement 1");
                {
                    System.out.println("statement 2");
                    break label1;
                }
            }

        }
        System.out.println("statement 4");

```
输出：
```bash
statement 1
statement 2
statement 4
```
可见，循环体其实是有一个隐式的标签，break直接退出循环，即为立刻结束离break最近的这一层隐式标签；同理，continue即为，从这层隐式标签开始的地方重新开始执行。
# 3 补充
```java
        label1:
        {

        }
        {
            while (true) {
                System.out.println("statement 1");
                {
                    System.out.println("statement 2");
                    break label1; //IDEA直接标红，说明检测到这个break动作不在任何被标记的代码块内
                }
            }

        }
        System.out.println("statement 4");
```
因此，不在被隐式/显式标记的代码块内执行break是不符合语法规范的。
