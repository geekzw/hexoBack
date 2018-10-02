---
title: 广度优先搜索(BFS)
date: 2016-12-14 09:47:21
categories: algorithm
tags: algorithm
---
广度优先搜索算法（Breadth-First-Search），又译作宽度优先搜索，或横向优先搜索，简称BFS，是一种图形搜索算法。简单的说，BFS是从根节点开始，沿着树的宽度遍历树的节点。
<!-- more -->
## 简介
广度优先搜索算法（Breadth-First-Search），又译作宽度优先搜索，或横向优先搜索，简称BFS，是一种图形搜索算法。简单的说，BFS是从根节点开始，沿着树的宽度遍历树的节点。如果所有节点均被访问，则算法中止。广度优先搜索的实现一般采用open-closed表。时间复杂度O(|V| + |E|),v是节点数，e是边数。应为广搜是地毯式搜索，所以不适合非常大的问题
## 分析
4 4
1 1 1 1
1 0 1 0
0 1 1 1
0 0 1 1
上面是一个4*4的，1代表通路，0代表不同路的迷宫，终点为右下
广搜面对这种图的时候，先从0，0点开始，然后把与0，0点相邻的，可以走通的点加入到队列中，然后再从队列中取出一个节点，再把与这个节点相邻的加入到队列中，如此往复，中间取出节点时判断是否到达终点，到达就退出（我们说的是有没有路径）。如果最终队列为空了，那就是没找到。队列我们都知道，先进先出原则。其实看有没有路径，深搜更合适，广搜适合找有多少条路径。
```java
public class BFS {
    public static int[][] dir = new int[][]{{-1,0,1,0},{0,1,0,-1}};
    static boolean[][] flag = new boolean[4][4];
    static boolean hasPath;

    public static void main(String[] args){
        int[][] maze = new int[][]{{1,1,1,1},{1,0,1,0},{0,1,1,1},{0,0,1,1}};
        boolean[][] flag = new boolean[4][4];
        bfs(maze);
        System.out.print(hasPath);
    }

    public static void bfs(int[][] maze){
        Queue<Node> queue = new LinkedList<Node>();
        boolean[][] flag = new boolean[4][4];
        Node node = new Node(0,0,maze[0][0]);
        flag[0][0] = true;
        queue.add(node);
        while(queue.size()>0){
            node = queue.poll();
            if(node.x == 3 && node.y == 3){
                hasPath = true;
                break;
            }
            for(int i=0;i<4;i++){
                int xx = node.x+dir[0][i];
                int yy = node.y+dir[1][i];
                if(xx>=0 && xx < maze.length && yy>=0 && yy< maze[xx].length && maze[xx][yy] == 1 && !flag[xx][yy]){
                    node = new Node(xx,yy,maze[xx][yy]);
                    flag[xx][yy] = true;
                    queue.offer(node);
                }
            }


        }

    }

    static class Node{
        int x;
        int y;
        int val;
        boolean visitied;

        public Node(int x,int y,int val){
            this.x = x;
            this.y = y;
            this.val = val;
        }
    }
}
```
前面讲过深搜的递归方式，非递归没有说。我们知道，递归方式就是方法不断入栈的过程。那么如果用非递归，其实就是我们自己维护一个栈，跟这个广搜的思想差不多，只不过一个是维护队列，一个是维护栈
