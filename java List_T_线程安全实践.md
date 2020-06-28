测试相同对象不同引用的锁，是否同一把锁。    //试验证明，无论引用名是什么，只要指向同一内存区域，锁的都是同一实例！！！！！

加锁时，同步块的的范围十分讲究！！！！！尤其是和迭代器相关的，从迭代器出现时，就必须要放进同步块里了。

ynchronized (list){
//这种写法，一旦拿到锁，就不会释放了，因为while（true）永远不会跳出，不会把锁让出去，因此，尽可能把缩小同步块范围，给别的线程留时间。
            while (true){
                list.add(String.valueOf(System.currentTimeMillis()));
                System.out.println(Thread.currentThread().getName());
            }
        }


list线程安全方法：
1、使用vector，而不是arraylist
2、再封装一层，用synchronizd让add remove等list相关操作具有互斥性
3、使用lock锁
4、直接使用synchronized将相关危险操作包裹，尤其注意迭代器相关操作。

