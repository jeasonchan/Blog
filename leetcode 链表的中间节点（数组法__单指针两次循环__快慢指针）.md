```java
package com.xxx.链表的中间节点;


import java.util.ArrayList;
import java.util.List;

/*
876. 链表的中间结点

给定一个带有头结点 head 的非空单链表，返回链表的中间结点。

如果有两个中间结点，则返回第二个中间结点。



示例 1：

输入：[1,2,3,4,5]
输出：此列表中的结点 3 (序列化形式：[3,4,5])
返回的结点值为 3 。 (测评系统对该结点序列化表述是 [3,4,5])。
注意，我们返回了一个 ListNode 类型的对象 ans，这样：
ans.val = 3, ans.next.val = 4, ans.next.next.val = 5, 以及 ans.next.next.next = NULL.

示例 2：

输入：[1,2,3,4,5,6]
输出：此列表中的结点 4 (序列化形式：[4,5,6])
由于该列表有两个中间结点，值分别为 3 和 4，我们返回第二个结点。



提示：

给定链表的结点数介于 1 和 100 之间。

数组法

单指针法


快慢指针法
 */
public class Main {
    //数组法
    public static ListNode middleNode_array(ListNode head) {
        List<ListNode> list = new ArrayList<>();
        while (null != head) {
            list.add(head);
            head = head.next;
        }
        int length = list.size();
        int index = length / 2;//整数相除，想下向下取整
        return list.get(index);
    }

    //单指针法
    public static ListNode middleNode_singlePoint(ListNode head) {
        int nodesNumber = 0;
        ListNode originHead = head;
        while (null != head) {
            nodesNumber++;
            head = head.next;
        }
        int targetIndex = nodesNumber / 2;

        ListNode target = originHead;

        //注意退出条件，多用边界条件试一试
        while (targetIndex > 0) {
            targetIndex--;
            target = target.next;

        }
        return target;
    }

    //快慢指针
    public static ListNode middleNode_fast_slow_point(ListNode head) {
        ListNode fastNode = head;
        ListNode slowNode = head;

        //加入第三个判断条件  是为了防止链表过短造成空指针异常，多用边界条件试验判断条件
        while (null != fastNode && null != slowNode && fastNode.next != null) {
            slowNode = slowNode.next;
            fastNode = fastNode.next.next;
        }

        return slowNode;
    }

}

class ListNode {
    int val;
    ListNode next;

    ListNode(int x) {
        val = x;
    }
}

```
