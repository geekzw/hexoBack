---
title: 深度优先搜索(DFS)
date: 2016-12-13 10:26:01
categories: algorithm
tags: algorithm
---
深度优先搜索解决的问题是联通图找路径或是判断连通性。解决问题的思想是从某一节点触发，一路向前，直到走不动了，才回退，再找其他路径，是不是很像不撞南墙不回头。下面给个迷宫的简单例子，分析下过程。
<!-- more -->
## 简介
深度优先搜索解决的问题是联通图找路径或是判断连通性。解决问题的思想是从某一节点触发，一路向前，直到走不动了，才回退，再找其他路径，是不是很像不撞南墙不回头。下面给个迷宫的简单例子，分析下过程。
4 4
1 1 1 1
1 0 1 0
0 1 1 1
0 0 1 1
第一行代表行数和列数，下面就是各行各列。1代表通路，0代表不通，问能不能找到到达终点（右下角）的路径。这个是超简单的深搜问题了，可以说直接无脑搜就行了。很明显是有通路的(0,0)->(0,1)->(0,2)->(1,2)->(2,2)->(2,3)->(3,3)。当然还有一条，不过看一条就够了。假如开始在左上角，每次优先向右走（这个优先也是随机的，往哪都行）。那么它会一直走到(0,3)再往前没路了，那么就回退到(0,2),因为走过(0,2)->(0,3)了，所以要换个方向走，换往下走吧就到了(1,2)。就照这个思路，走不动了就回退一格，换方向走，最终总能找到那条可以走通的路（如果存在的话）
```java
public class DFS {
    //每个方向上x，y变化值
    public static int[][] dir = new int[][]{{-1,0,1,0},{0,1,0,-1}};
    static boolean[][] flag = new boolean[4][4];//标记这个路走过没有
    static boolean hasPath;//是否找到了通路

    public static void main(String[] args){
        //迷宫
        int[][] maze = new int[][]{{1,1,1,1},{1,0,1,0},{0,1,1,1},{0,0,1,1}};//
        flag[0][0] = true;
        dfs(maze,0,0);
        System.out.print(hasPath);

    }

    public static void dfs(int[][]maze,int x,int y ){
        if(x == 3 && y == 3){//找到了
            hasPath = true;
        }
        //换方向
        for(int i=0;i<4;i++){
            int xx = x+dir[0][i];
            int yy = y+dir[1][i];
            //条件虽多，但是都很好理解，坐标不能越界，节点没有访问过，节点可以走通
            if(xx>=0 && xx < maze.length && yy>=0 && yy< maze[xx].length && maze[xx][yy] == 1 && !flag[xx][yy]){
                flag[xx][yy] = true;
                dfs(maze,xx,yy);
                flag[xx][yy] = false;
            }
        }

    }
}

```
