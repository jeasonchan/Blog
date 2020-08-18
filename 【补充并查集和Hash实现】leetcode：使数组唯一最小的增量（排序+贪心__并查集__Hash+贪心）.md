# 1 排序+贪心
```java
package com.xxx.使数组唯一最小的增量;

import java.util.Arrays;

/*
给定整数数组 A，每次 move 操作将会选择任意 A[i]，并将其递增 1。

返回使 A 中的每个值都是唯一的最少操作次数。

示例 1:

输入：[1,2,2]
输出：1
解释：经过一次 move 操作，数组将变为 [1, 2, 3]。

示例 2:

输入：[3,2,1,2,1,7]
输出：6
解释：经过 6 次 move 操作，数组将变为 [3, 4, 1, 2, 5, 7]。
可以看出 5 次或 5 次以下的 move 操作是不能让数组的每个值唯一的。

提示：

    0 <= A.length <= 40000
    0 <= A[i] < 40000


 */

public class Main {
    /*
    leetcode 945
     */
    public static int minIncrementForUnique(int[] A) {
        int result = 0;

        //先进行从小到大的排序
        Arrays.sort(A);//如果使用冒泡排序，时间会超时

//        for (int i = 0; i < A.length; i++) {
//            for (int j = i + 1; j < A.length; j++) {
//                if (A[j] < A[i]) {
//                    int temp = A[i];
//                    A[i] = A[j];
//                    A[j] = temp;
//
//
//                }
//            }
//        }
        System.out.println("after resort:" + Arrays.toString(A));

        //使用贪心算法，求局部最优解
        //通过for循环，从长度为0的局部最优开始，逐渐拓展到全长度的最优
        //下一个的最优，基于前一个最优的决定
        for (int i = 1; i < A.length; i++) {
            //局部最优的解法是，最后一个数字做到比倒数第二个数字至少大1即可

            //因为排序过，只需要解决最后两位相等的情况;还要解决，前面的数组增加过后比当前数字大的情况
            if (A[i] <= A[i - 1]) {
                int oldValue = A[i];
                System.out.println("before change:" + Arrays.toString(A));
                A[i] = A[i - 1] + 1;
                System.out.println("after change:" + Arrays.toString(A));
                result = result + (A[i] - oldValue);
            }


        }


        return result;
    }


    public static void main(String[] args) {
//        int[] A = new int[]{3, 2, 1, 2, 1, 7};
        int[] A = new int[]{1, 0};

        System.out.println(minIncrementForUnique(A));
    }


}

```

# 2 Hash+贪心

# 3 并查集
