# 1 hash容器原理


# 2 hashcode和equals方法
**重写equals需要重写hashcode**

对于自定义类，如果不重写equals方法，equals会默认直接比较内存地址。如果我们想比较内容的话就需要在类里面重写equals方法和hashcode方法，这两个方法是属于Object里面的方法，任何类都隐式的继承了Object类。

那为什么非得重写hashcode呢？因为对象的比较有硬性规定：
1. 应用执行期间，同一个对象内容不发生改变，经过多次调用，hashCode方法都必须始终返回同一个值。如果把对象重新copy到另外一个应用程序里，hashcode可以和上一个的不一样。
2. 如果两个对象根据equals(Object)方法比较是相等的，那么所产生的的hashcode必须一样。
3. 如果两个对象根据equals(Object)方法比较是不相等的，那么产生的hashcode可能一样，也可能不一样。

向基于hash的hashSet里放对象的时候，先算取hashcode值，然后会定位到该对象所在的桶，然后再用equals比较内容，发现内容也一样，于是新对象会替换就旧对象（对于java，放的其实是对象的内存地址，放的是引用）。

注意：累对象里属性会很多，每次属性会变动，我们都需要改动equals方法，可是还要保证上面的第二条原则，那自然是要重写hashcode的。
