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

