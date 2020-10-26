# 1 背景
闲来无事看看insert的源码，注解写着vector会在position的before位置插入数据，同时该方法比较消耗性能，插入操作较多的建议使用List数据容器，推测是每次插入可能都要进行的扩容、数据迁移操作，也就是和JAVA中hashmap相似，如果hashmap一开始初始化时不定义大小，会在之后插入数据时频繁扩容、迁移数据，影像性能。经过试验和源码，发现确实如此。


# 2 测试代码
```cpp

#include "./Solution.hpp"
#include <vector>
#include <iostream>
#include <iterator>

int main() {
    using namespace leetcode143;
    using namespace std;

    vector<ListNode> vector1;
    ListNode *head = new ListNode(0);

    vector1.emplace_back(1, head);
//    vector1.push_back(bool(2, head));
    vector1.emplace_back(2, head);
    vector1.emplace_back(3, head);
    vector1.emplace_back(4, head);


    /*
     * 运行结果显示vector de di zhi bu bian,
     * shou yuan su de di zhi yi jing bian hua,
     * nan guai insert shi yi yi ge hao shi cao zuo
     */
    cout << "size=" << vector1.size() << endl;
    cout << "capacity=" << vector1.capacity() << endl;
    cout << "&vector1=" << &vector1 << endl;
    cout << "&vector1[0]=" << &vector1[0] << endl;

    vector<ListNode>::iterator it = vector1.begin();

    while (it != vector1.end()) {
        cout << it->val << endl;
        ++it;
    }

    cout << "===========================\n";

    it = vector1.begin();

    vector1.insert(it, ListNode(5, head));

    cout << "size=" << vector1.size() << endl;
    cout << "capacity=" << vector1.capacity() << endl;
    cout << "&vector1=" << &vector1 << endl;
    cout << "&vector1[0]=" << &vector1[0] << endl;
    cout << "&vector1[1]=" << &vector1[1] << endl;

    it = vector1.begin();
    while (it != vector1.end()) {
        cout << it->val << endl;
        ++it;
    }

    /*  输出：
        size=4
        capacity=4
        &vector1=0x7ffd04af1da0
        &vector1[0]=0x12f1ee0
        1
        2
        3
        4
        ===========================
        size=5
        capacity=8
        &vector1=0x7ffd04af1da0
        &vector1[0]=0x12f2340
        &vector1[1]=0x12f2350
        5
        1
        2
        3
        4

     */
    return 0;
}
```
从书出可以看出，经过insert操作，vector的capcacity已经是8了，并且元素的地址也发生了变化（如果不发生数据迁移，新向量的第二个元素的地址应该和旧向量的第一个元素的地址相同），但vector自身的地址没变，来看看insert的源码究竟干了什么。


# 3 insert源码解析
擦，centos下的源码到：
```cpp
iterator
_M_insert_rval(const_iterator __position, value_type&& __v);
```
就没了……回家看看win下面的实现……




