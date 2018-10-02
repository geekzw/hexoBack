---
title: 最小生成树之Prim
date: 2016-12-10 14:03:45
categories: algorithm
tags: algorithm
---
普里姆算法（Prim算法），图论中的一种算法，可在加权连通图里搜索最小生成树。意即由此算法搜索到的边子集所构成的树中，不但包括了连通图里的所有顶点（英语：Vertex (graph theory)），且其所有边的权值之和亦为最小。该算法于1930年由捷克数学家沃伊捷赫·亚尔尼克（英语：Vojtěch Jarník）发现；并在1957年由美国计算机科学家罗伯特·普里姆（英语：Robert C. Prim）独立发现；1959年，艾兹格·迪科斯彻再次发现了该算法。因此，在某些场合，普里姆算法又被称为DJP算法、亚尔尼克算法或普里姆－亚尔尼克算法。
<!-- more -->
## Prim
### 概述
普里姆算法（Prim算法），图论中的一种算法，可在加权连通图里搜索最小生成树。意即由此算法搜索到的边子集所构成的树中，不但包括了连通图里的所有顶点（英语：Vertex (graph theory)），且其所有边的权值之和亦为最小。该算法于1930年由捷克数学家沃伊捷赫·亚尔尼克（英语：Vojtěch Jarník）发现；并在1957年由美国计算机科学家罗伯特·普里姆（英语：Robert C. Prim）独立发现；1959年，艾兹格·迪科斯彻再次发现了该算法。因此，在某些场合，普里姆算法又被称为DJP算法、亚尔尼克算法或普里姆－亚尔尼克算法。
### 简介
prim解决的是加权连通图的最小集合（无向图），思想很简单。时间复杂度n^2
* 再图中任意选一点作为源点
* 找据源点最近的点，加入到源点所在的集合中
* 如此往复循环，知道源点所在的集合包含了所有顶点
就是这么简单，没有那么多的弯弯道道。下面上代码
```java
public class Prim {
    //main方法中主要做了初始化工作，跟算法没什么关系
    public static void main(String[] args){

        int pointCount;
        int pathCount;
        int[][] pathSet;
        Scanner cin = new Scanner(System.in);
        pointCount = cin.nextInt();
        pathCount = cin.nextInt();
        pathSet = new int[pointCount+1][pointCount+1];
        for(int i=1;i<=pointCount;i++){
            for(int j=1;j<=pointCount;j++){
                pathSet[i][j] = Integer.MAX_VALUE;
            }
        }

        for(int i=0;i<pathCount;i++){
            int a = cin.nextInt();
            int b = cin.nextInt();
            int c = cin.nextInt();
            pathSet[a][b] = pathSet[b][a] = c;
        }
        prim(pathSet);
    }
    //prim算法核心
    public static void prim(int[][] pathSet){
        int[] minPointSet = new int[pathSet.length];//最小生成树集合
        boolean[] isVisited = new boolean[pathSet.length];//判断是否已经在集合中
        int minPathLenth = 0;//最小生成树目前的长度
        int visiIndex = 0;//当前位置加入的节点
        isVisited[1] = true;
        minPointSet[++visiIndex] = 1;
        while(visiIndex!= pathSet.length-1){
            int minLengh = Integer.MAX_VALUE;
            int currentIndex = 1;
            //在生成树集合和待选集合中选出连接两方的最短距离
            for(int i=1;i<=visiIndex;i++){//生成树集合阵营中选出一个点
                for(int j=1;j<pathSet.length;j++){//待选阵营中选出一个点
                    if(!isVisited[j] && pathSet[minPointSet[i]][j]<minLengh){
                        minLengh = pathSet[minPointSet[i]][j];
                        currentIndex = j;
                    }
                }
            }
            isVisited[currentIndex] = true;//选出后加入到生成树集合
            minPointSet[++visiIndex] = currentIndex;
            minPathLenth+=minLengh;
        }

        for(int i=1;i<=visiIndex;i++){
            System.out.print(minPointSet[i]+"");
        }
        System.out.print("\n"+minPathLenth);
    }
}

```
