@[TOC]
# 1 原题描述
返回字符串 text 中按字典序排列最小的子序列，该子序列包含 text 中所有不同字符一次。
# 2 
```java
class Solution {
    public  String smallestSubsequence(String text) {
        int len = text.length();
        char tmp;
        StringBuffer sb = new StringBuffer();
        Stack<Character> stack = new Stack<>();
        for (int i = 0; i < len; i++) {
            tmp = text.charAt(i);
            if (stack.contains(tmp))
                continue;
            while (!stack.isEmpty() && tmp < stack.peek() && text.lastIndexOf(stack.peek()) > i) {
                stack.pop();
            }
            stack.push(tmp);
        }
        while (!stack.isEmpty())
            sb.append(stack.pop());
        return sb.reverse().toString();
    }
}
```
