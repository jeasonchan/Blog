# 1 背景

参考文章：

[BFS和DFS详解及Java实现](https://www.cnblogs.com/developerY/p/3323264.html)

待补充项目：

1. 遍历到的完整路径输出
2. 补充相应的算法实际题目



# 2 图的表示
邻接表和邻接矩阵法

# 3 BFS
遍历所有的节点，并打印某节点到起始点的举例：
```java
package com.jeasonchan.BFS广度优搜索;

import java.util.HashMap;
import java.util.LinkedList;
import java.util.Map;
import java.util.Queue;

public class Main {
    public static void main(String[] args) {
        /*
        图：
        a:b d e
        b:a c
        c:b f d
        d:c a e
        e:a d
        f:c g
        g:f

         */

        //构造图
        Map<String, String[]> node_Neighbour_map = new HashMap<>();
        node_Neighbour_map.put("a", new String[]{"b", "d", "e"});
        node_Neighbour_map.put("b", new String[]{"a", "c"});
        node_Neighbour_map.put("c", new String[]{"b", "f", "d"});
        node_Neighbour_map.put("d", new String[]{"c", "a", "e"});
        node_Neighbour_map.put("e", new String[]{"a", "d"});
        node_Neighbour_map.put("f", new String[]{"c", "g"});
        node_Neighbour_map.put("g", new String[]{"f"});

        //调用方法打印每个节点举例触发点的举例
        pringEachNodeDistance("a", node_Neighbour_map);


    }

    /**
     * 打印每个节点距离起点的距离
     *
     * @param startNode
     * @param graph
     */
    public static void pringEachNodeDistance(String startNode, Map<String, String[]> graph) {
        Queue<String> queue = new LinkedList<>();
        Map<String, Integer> record = new HashMap<>();

        int distance = 0;
        record.put(startNode, distance);
        queue.add(startNode);


        while (!queue.isEmpty()) {
            String topNode = queue.poll();

            /*
            有时候为了寻找每一个特殊的节点，可在此处判断，找到了跳出循环即可
             */

            System.out.println("from " + startNode + " to " + topNode + " , distance is " + record.get(topNode));

            String[] nextNodes = graph.get(topNode);

            //遍历相邻的节点
            for (String each : nextNodes) {

                //只需要处理未记录的节点
                if (!(record.containsKey(each))) {
                    record.put(each, record.get(topNode) + 1);//记录距离，用来标记该点是已检查过
                    queue.add(each);//加入队列
                }
            }


        }

    }
}

```

## 3.1 BFS完整路径输出

（见本章  5 实现路径记录功能）



# 4 DFS

```java
package default_package.BFS和DFS;

import java.util.Arrays;
import java.util.Map;

public class DFS {
    /*
    DFS  主要用来寻找，以某个点为起点时，所有的完整的、有终点的路径情况
     */

    public static void listAllRoutesFromStartNode(Map<String, String[]> graph, Map<String, Boolean> record, String startNode) {
        if (!record.containsKey(startNode)) {
            record.put(startNode, true);

            System.out.println("node:" + startNode);


            String[] neighbourNodes = graph.get(startNode);

            //所连接的节点都已经被访问过一遍了，则说明要返回父级节点重新访问
            //也就说明，当前节点是一个末端节点
            if (record.keySet().containsAll(Arrays.asList(neighbourNodes))) {
                System.out.println("return the previous node");
            }

            for (String each : neighbourNodes) {
                listAllRoutesFromStartNode(graph, record, each);
            }


        }
    }
}

```

测试及输出：

```java
public class Main {
    public static void main(String[] args) {
        //构造一个的图
        /*
        图：
        a:b d e
        b:a c
        c:b f d
        d:c a e
        e:a d
        f:c g
        g:f

         */

        //构造图
        Map<String, String[]> node_Neighbour_map = new HashMap<>();
        node_Neighbour_map.put("a", new String[]{"b", "d", "e"});
        node_Neighbour_map.put("b", new String[]{"a", "c"});
        node_Neighbour_map.put("c", new String[]{"b", "f", "d"});
        node_Neighbour_map.put("d", new String[]{"c", "a", "e"});
        node_Neighbour_map.put("e", new String[]{"a", "d"});
        node_Neighbour_map.put("f", new String[]{"c", "g"});
        node_Neighbour_map.put("g", new String[]{"f"});

//        BFS.printEachNodeDistance(node_Neighbour_map, "a");


        Map<String, Boolean> record = new LinkedHashMap<>();
        DFS.listAllRoutesFromStartNode(node_Neighbour_map, record, "a");

        System.out.println(record);


    }
}


/*
node:a
node:b
node:c
node:f
node:g
return the previous node
node:d
node:e
return the previous node
{a=true, b=true, c=true, f=true, g=true, d=true, e=true}
*/
```

## 4.1 DFS完整路径输出

（见本章  5 实现路径记录功能）



# 5 BFS和DFS的路径记录实现

**尤其要注意！！！！！！！DFS中，达到边缘后，就开始向上逐渐退出一层迭代，有记录路径操作时，务必！！！！！！！！！在退出一层递归时，同时回退对路径记录的修改！！**

**BFS选择的路径记录数据结构，一定要注意避免对父节点路径记录对象的修改，比如，new StingBuilder()并不会进行深拷贝！！！！**

```java
package default_package.括号生成_DFS_路径记录;

import java.util.*;
import java.util.stream.Collectors;


/*
数字 n 代表生成括号的对数，请你设计一个函数，用于能够生成所有可能的并且 有效的 括号组合。



示例：

输入：n = 3
输出：[
       "((()))",
       "(()())",
       "(())()",
       "()(())",
       "()()()"
     ]

来源：力扣（LeetCode）
链接：https://leetcode-cn.com/problems/generate-parentheses
著作权归领扣网络所有。商业转载请联系官方授权，非商业转载请注明出处。


先强调几个事实：
尝试去生成一个有效的括号串时，如果：
1、剩下的做括号个数大于0，可以在前面括号串的基础上再加上一个左括号
2、如果，剩余的右括号<左括号，也就是 字符串中的  右括号>左括号   了，显然是不合理的，无论怎么加括号，该字符串都不会是有效括号串了
3、如果 剩余的 右括号>左括号 ，也就是 字符串中的  右括号<左括号  ，才有可能添加右括号

综上，如果，将添加括号的过程一个生成二叉树的过程，添加子节点生成条件，就有两种情况：
1、剩余左括号个数>0，添加左括号，生成左儿子
2、剩余的右括号>左括号，添加右括号，生成右儿子

如何避免生成重复的字符串，从二叉树的生成过程可以看出，如果是一样的字符串，那就是二叉树的路径会发生完全重合，显然不可能
因此，按照以上两个规则生成二叉树时，就自然完成了去重流程

解题步骤：
（1） 先按照规则生括号二叉树
（2）DFS搜索出末端节点，即为最终的结果
注：生成二叉树的过程，其实就可以看作是 DFS的过程，可以将2步合成1步


编码实现要点：
记录路径时，每从一层递归中退出，都要回退对路径记录的修改！！！！！

 */
class Solution {
    static enum RouteGenerateType {
        BFS("BFS"), DFS("DFS");

        RouteGenerateType(String value) {
            this.value = value;
        }

        private String value;
    }


    public List<String> generateParenthesis(int n, RouteGenerateType routeGenerateType) {
        Node root = new Node(Node.NodeType.LEFT);
        generateTree(n - 1, n, root);

        List<String> result = new ArrayList<>();

        switch (routeGenerateType) {
            case BFS:
                //使用BFS获取路径
                result = generateRoute_bfs(root, n);
                break;
            case DFS:
                //使用DFS获取路径
                generateRoute_dfs(root, new StringBuilder(), result);
                break;

        }

        return result;
    }

    /**
     * 通过深度优先搜素，得到末端节点，并输出记录下的遍历路径
     *
     * @param currentNode
     * @param route
     * @param result
     */
    private void generateRoute_dfs(Node currentNode, StringBuilder route, List<String> result) {
        StringBuilder newRoute = route.append(currentNode.value);

        if (null == currentNode.leftNode && null == currentNode.rightNode) {
            //说明当前是末端节点
            result.add(newRoute.toString());
            System.out.println(newRoute);
            return;
        }

        if (null != currentNode.leftNode) {
            generateRoute_dfs(currentNode.leftNode, newRoute, result);
            newRoute.delete(newRoute.length() - 1, newRoute.length());//回退到上一层时，同时回退对路径的修改
        }

        if (null != currentNode.rightNode) {
            generateRoute_dfs(currentNode.rightNode, newRoute, result);
            newRoute.delete(newRoute.length() - 1, newRoute.length());//回退到上一层时，同时回退对路径的修改
        }


    }

    /*
    通过广度优先遍历时，BFS本身的终止条件是Queue中没有节点需要遍历了，
    但是，对于放入队列的节点，我们可以进行有条件的筛选，
    对于本体，我们的目的是遍历得到末端节点，因此，可以不必限制条件，让其自然结束；
    也可以，只加入距离<=2n-1的节点，因为括号字符串的长度是确定的2n，末端节点距离root节点的距离必定是2n-1
     */
    private List<String> generateRoute_bfs(Node root, int n) {
        Map<Node, String> record = new HashMap<>();
        //此处使用的Node没有重写equals和hashcode，会直接使用内存地址，正好可以区分开不同的左右括号节点

        Queue<Node> queue = new LinkedList<>();
        queue.add(root);
        record.put(root, root.value);


        while (!queue.isEmpty()) {
            Node currentNode = queue.poll();
            String currentRoute = record.get(currentNode);


            //一般情况下，都要先判断子节点有没有在record中出现过，但是，因为是二叉树，不是图，不会重复访问，略去record判断
            if (null != currentNode.leftNode) {
                queue.add(currentNode.leftNode);

                //务必注意！！！！！！！！！！！！！！！！！放进record的必须和前面的currentNode脱离关系，要不然会持续影响后面的
                //new StringBuilder()  并不会进行深拷贝，会直接影响前后对象
                record.put(currentNode.leftNode, currentRoute + currentNode.leftNode.value);
            }

            if (null != currentNode.rightNode) {
                queue.add(currentNode.rightNode);
                record.put(currentNode.rightNode, currentRoute + currentNode.rightNode.value);
            }


        }


        List<String> result = new ArrayList<>();
        record.values().stream().filter((each) -> each.length() == 2 * n).forEach((each) -> {
            System.out.println(each);
            result.add(each);
        });


        return result;
    }


    private void generateTree(int remainedLeft, int remainedRight, Node currentNode) {
        if (0 == remainedLeft && 0 == remainedRight) {
            return;
        }
        if (remainedLeft > 0) {
            //添加左儿子
            currentNode.leftNode = new Node(Node.NodeType.LEFT);
            generateTree(remainedLeft - 1, remainedRight, currentNode.leftNode);

        }
        if (remainedRight > remainedLeft) {
            //添加右儿子
            currentNode.rightNode = new Node(Node.NodeType.RIGHT);
            generateTree(remainedLeft, remainedRight - 1, currentNode.rightNode);
        }

    }


    //二叉树节点
    static class Node {
        public enum NodeType {
            LEFT("("),
            RIGHT(")");

            NodeType(String value) {
                this.value = value;
            }

            public String value;
        }


        Node(NodeType nodeType) {
            this.value = nodeType.value;
        }

        public String value;
        public Node leftNode = null;
        public Node rightNode = null;

        @Override
        public String toString() {
            return "Node{" +
                    "value='" + value + '\'' +
                    ", leftNode=" + leftNode +
                    ", rightNode=" + rightNode +
                    '}';
        }
    }
}

```

调用：

```java
package default_package.括号生成_DFS_路径记录;

import default_package.BFS和DFS.BFS;

public class Main {
    public static void main(String[] args) {
        System.out.println("BFS:");
        new Solution().generateParenthesis(3, Solution.RouteGenerateType.BFS);


        System.out.println("DFS:");
        new Solution().generateParenthesis(3, Solution.RouteGenerateType.DFS);
    }
}

```

