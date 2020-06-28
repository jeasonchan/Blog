# 1 hash容器原理（冲突解决等）
[hashmap冲突的解决方法以及原理分析](https://www.cnblogs.com/peizhe123/p/5790252.html)
[Map之HashMap的get与put流程，及hash冲突解决方式](https://www.cnblogs.com/many-object/p/8909846.html)

查找：先调用对象的hashCode方法获取该对象的hash值，作为在hash表中的索引，根据索引找到的对象的引用，根据引用找到对象，再调用对象的equals方法判断是否相等。

HashMap中，put(key,value)时，如果的hash(key)算出的索引对应的位置已经有对象的，但是，key不是同一个key（也就是，不同的key，算出的哈希值相同，key的hashCode函数有问题……），就发生冲突了；HashMap冲突的解决方法是，生成一个Entry链表，越靠后进来的，就越在Entry链的前端。

其余的冲突解决方法还有：
开发地址法

## 1.1 冲突后的get
当调用get方法时
```java
 public V get(Object key) {
        Node<K,V> e;
        return (e = getNode(hash(key), key)) == null ? null : e.value;
    }
```
**会通过key的hash值获取table中的Node，并通过key的equals方法来确定要取的Node，在返回Node的value值。**

通过HashMap的存储结构可以发现当我们遍历HashMap时通过entrySet方法性能会高一点，因为它直接返回了存储的Map.Entry类，而遍历key方式是通过遍历Map.Entry取出key，我们在调用get(key)方法时又会去取一次Entry，所以性能会比较低


# 2 hashcode和equals方法
## 2.1 hashCode()
hashCode() 的作用是获取哈希码，也称为散列码；它实际上是返回一个int整数。这个哈希码的作用是确定该对象在哈希表中(哈希表，就是   hash(key)--Entry<key,value> 的表 )的索引位置。hashCode() 定义在JDK的Object.java中，这就意味着Java中的任何类都包含有hashCode() 函数。

不重写的话，就是直接对对象的内存地址计算哈希值。

## 2.2 equals()
不重写的话，就是直接比较对象的内存地址。



## 2.3 两者关联
**重写equals需要重写hashcode**

对于自定义类，如果不重写equals方法，equals会默认直接比较内存地址。如果我们想比较内容的话就需要在类里面重写equals方法，并同时重写hashcode方法，这两个方法是属于Object里面的方法，任何类都隐式的继承了Object类。

那为什么非得重写hashcode呢？因为对象的比较有硬性规定：
1. 应用执行期间，同一个对象内容不发生改变，经过多次调用，hashCode方法都必须始终返回同一个值。如果把对象重新copy到另外一个应用程序里，hashcode可以和上一个的不一样。
2. 如果两个对象根据equals(Object)方法比较是相等的，那么所产生的的hashcode必须一样。
3. 如果两个对象根据equals(Object)方法比较是不相等的，那么产生的hashcode可能一样，也可能不一样。

向基于hash的hashSet里放对象的时候，先算取hashcode值，然后会定位到该对象所在的桶，然后再用equals比较内容，发现内容也一样，于是新对象会替换就旧对象（对于java，放的其实是对象的内存地址，放的是引用）。

注意：累对象里属性会很多，每次属性会变动，我们都需要改动equals方法，可是还要保证上面的第二条原则，那自然是要重写hashcode的。

## 2.3.1 有哪些覆写hashCode的诀窍
**一个好的hashCode的方法的目标：为不相等的对象产生不相等的散列码，同样的，相等的对象必须拥有相等的散列码。**

1. 把某个非零的常数值，比如17，保存在一个int型的result中；
2. 对于每个关键域f（equals方法中设计到的每个域），作以下操作：

a.为该域计算int类型的散列码；

i.如果该域是boolean类型，则计算(f?1:0),

ii.如果该域是byte,char,short或者int类型,计算(int)f,

iii.如果是long类型，计算(int)(f^(f>>>32)).

iv.如果是float类型，计算Float.floatToIntBits(f).

v.如果是double类型，计算Double.doubleToLongBits(f),然后再计算long型的hash值

vi.如果是对象引用，则递归的调用域的hashCode，如果是更复杂的比较，则需要为这个域计算一个范式，然后针对范式调用hashCode，如果为null，返回0

vii. 如果是一个数组，则把每一个元素当成一个单独的域来处理。

b.result = 31 * result + c;

3、返回result

4、编写单元测试验证有没有实现所有相等的实例都有相等的散列码。

给个简单的例子：

```java
@Override
public int hashCode() {  
  int result = 17;  
  result = 31 * result + name.hashCode();  
  return result;
}
```
这里再说下2.b中为什么采用31*result + c,乘法使hash值依赖于域的顺序，如果没有乘法那么所有顺序不同的字符串String对象都会有一样的hash值，而31是一个奇素数，如果是偶数，并且乘法溢出的话，信息会丢失，31有个很好的特性是31*i ==(i<<5)-i,即2的5次方减1，虚拟机会优化乘法操作为移位操作的。


