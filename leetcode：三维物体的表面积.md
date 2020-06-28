# 1 概要
题目本身没什么技术含量，在优化方法中，使用两个固定数组和一个for循环替代原先的四个if判断，更加简洁


# 2 代码
```java
package com.zte.三维物体的表面积;


/*
在 N * N 的网格上，我们放置一些 1 * 1 * 1  的立方体。

每个值 v = grid[i][j] 表示 v 个正方体叠放在对应单元格 (i, j) 上。

请你返回最终形体的表面积。

来源：力扣（LeetCode）
链接：https://leetcode-cn.com/problems/surface-area-of-3d-shapes
著作权归领扣网络所有。商业转载请联系官方授权，非商业转载请注明出处。


用例：
[[1,2],[3,4]]


[[1,0],[0,2]]
 */
public class Main {
    public int surfaceArea(int[][] grid) {
        int j_lemgth = grid[0].length;//
        int i_length = grid.length;
        int area = 0;


        for (int i = 0; i < i_length; i++) {
            for (int j = 0; j < j_lemgth; j++) {
                int height = grid[i][j];//当前单元格的高度

                //仅添加当前的水平方给总面积的贡献
                //只贡献一个顶面面积
                area += 0 == height ? 0 : 2;


                //变化j，看周围是否被重合
                if (j + 1 < j_lemgth) {
                    int height_j_1 = grid[i][j + 1];
                    area += height >= height_j_1 ? height - height_j_1 : 0;//只有自己比较高的时候，才能真正贡献表面积
                } else {
                    //已经是边界，周围无遮挡，直接贡献面积
                    area += height;
                }

                if (j - 1 >= 0) {
                    int height_1_j = grid[i][j - 1];
                    area += height >= height_1_j ? height - height_1_j : 0;//只有自己比较高的时候，才能真正贡献表面积
                } else {
                    //周围无遮挡，直接贡献面积
                    area += height;
                }


                //变化i，看周围是否重合
                if (i + 1 < i_length) {
                    int height_i_1 = grid[i + 1][j];
                    area += height >= height_i_1 ? height - height_i_1 : 0;//只有自己比较高的时候，才能真正贡献表面积
                } else {
                    //周围无遮挡，直接贡献面积
                    area += height;
                }

                if (i - 1 >= 0) {
                    int height_1_i = grid[i - 1][j];
                    area += height >= height_1_i ? height - height_1_i : 0;//只有自己比较高的时候，才能真正贡献表面积
                } else {
                    //周围无遮挡，直接贡献面积
                    area += height;
                }

            }
        }


        return area;
    }


    //========优化四方向拓展=========
    public int surfaceArea_with_4_directions_array(int[][] grid) {
        int[] dr = new int[]{0, 1, 0, -1};
        int[] dc = new int[]{1, 0, -1, 0};

        int N = grid.length;
        int ans = 0;

        for (int r = 0; r < N; ++r)
            for (int c = 0; c < N; ++c)
                if (grid[r][c] > 0) {
                    ans += 2;
                    for (int k = 0; k < 4; ++k) {

                        //向四个方向变换坐标
                        int nr = r + dr[k];
                        int nc = c + dc[k];

                        //拓展点的方块高度
                        int nv = 0;
                        if (0 <= nr && nr < N && 0 <= nc && nc < N)
                            nv = grid[nr][nc];

                        ans += Math.max(grid[r][c] - nv, 0);
                        //通过求最大值，替换了我的三元表达式   area += height >= height_j_1 ? height - height_j_1 : 0;
                    }
                }

        return ans;
    }

}
```
