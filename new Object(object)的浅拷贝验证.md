验证代码如下：
```java
    public static void main(String[] args) {
        ConcurrentLinkedQueue<Bean> queue1=new ConcurrentLinkedQueue<>();
        queue1.add(new Bean());

        ConcurrentLinkedQueue<Bean> queue2=new ConcurrentLinkedQueue<>(queue1);
        System.out.println(queue2);

        queue1.peek().setNumber(111);
        System.out.println(queue2);

        queue1.add(new Bean());
        System.out.println(queue2);
        
        //=========================================================
        ConcurrentLinkedQueue<String> queue3=new ConcurrentLinkedQueue<>();
        queue3.add("3333");
        queue3.add("4444");
        queue3.add("5555");

        ConcurrentLinkedQueue<String> queue4=new ConcurrentLinkedQueue<>(queue3);
        System.out.println(queue3);
        System.out.println(queue4);

        queue3.remove("4444");
        System.out.println(queue3);
        System.out.println(queue4);

    }
```
输出：
```shell
[Bean{haha='1', number=1}]
[Bean{haha='1', number=111}]
[Bean{haha='1', number=111}]
[3333, 4444, 5555]
[3333, 4444, 5555]
[3333, 5555]
[3333, 4444, 5555]
```
可见，新队列只是旧队列的浅拷贝，队列本身的“坑位”个数是和旧队列相同的，但是，“坑位”中的具体对象的，由于从旧队列取“坑位”中的内容时，只是通过“=”进行赋值操作，因此，若直接进行基本数据类型赋值，新旧队列不相干，若是操作对象赋值，由于只是引用赋值，本质上还是使用了旧队列中的对象，对旧队列中和新队列“共享”的对象的修改，会影响新队列。
如果想，对队列这样的集合进行深拷贝，可结合java8的流处理，或者使用字节流和对象流对对象进行深拷贝。
