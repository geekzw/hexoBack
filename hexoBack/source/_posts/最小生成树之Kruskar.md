---
title: 最小生成树之Kruskar
date: 2016-12-11 14:00:09
categories: algorithm
tags: algorithm
---
克鲁斯卡尔算法用与求解有权值的无相连通图的最小生成树。思想是先按照权值排序，然后每次把权值最小的节点连接起来，等所有节点都互相连接后，最小生成树就出来了。其中节点的链接，就是通过并查集实现的。
<!-- more -->
## 贪心算法
所谓贪心算法是指，在对问题求解时，总是做出在当前看来是最好的选择。也就是说，不从整体最优上加以考虑，他所做出的仅是在某种意义上的局部最优解。
贪心算法没有固定的算法框架，算法设计的关键是贪心策略的选择。必须注意的是，贪心算法不是对所有问题都能得到整体最优解，选择的贪心策略必须具备无后效性，即某个状态以后的过程不会影响以前的状态，只与当前状态有关。所以对所采用的贪心策略一定要仔细分析其是否满足无后效性。（听上去特别像dp有没有，其实dp就是贪心思想）
## 并查集
并查集思想用于求图的联通分量的个数等相关问题，能快速的合并和查询元素所在集合，用到这里很完美，先介绍下并查集的用法。并查集的核心操作
### init
```java
    for(int i=0;i<father.length;i++){ 
       father[i] = i;
     }  
```
father数组是存放父节点信息的，初始化的时候父节点就是自己，也就是所有节点单独为一个集合。
### search(int x)
##### 1.递归实现找祖先 
```java
  int search(int x){ 
    if (x != father[x]){ 
     father[x] = search(father[x]); 
    } 
    return father[x]; 
   } 
```
这是第一种查找祖先的方法，采用了递归的形式，入栈是为了找到自己的祖先，出栈（回朔）的时候把所有子节点的父节点都指向祖先，这样当下次再找祖先的时候，节约时间，并查集中叫：路径压缩
##### 1.循环实现找祖先 
```java
  nt search(int x){ 
     int k,root; 
     root=x; 
     while(root!=father[root])  //循环找x的根      
         root=father[root]; 
     while(x!=root)//本循环修改查找路径中的所有节点使其指向根节点---压缩 
     { 
         k=father[x]; 
         father[x]=root;//指向根节点 
         x=k; 
     } 
     return x; 
    } 
```
目的同上，只是实现方法不同，这个是先找到祖先，然后遍历子树，把子节点的父节点改成祖先节点
### union(x,y)
```java
  void union(int x, int y){
    x = Find_Set(x); 
    y = Find_Set(y); 
    if (x == y) return;
    father[y] = x;
} 
```
我这里只是简单的，把一个集合关联到另一个集合上，其实可以开个辅助数组，记录树的长度，把短的关联到长的上面，也是一种优化。
## Kruskar
说了半天，终于要说今天的主角了。克鲁斯卡尔算法用与求解有权值的无相连通图的最小生成树。思想是先按照权值排序，然后每次把权值最小的节点连接起来，等所有节点都互相连接后，最小生成树就出来了。其中节点的链接，就是通过并查集实现的。
```java

public class Kruskar {

    private static int[] father;
    private static int minPath;
    //初始化
    public static void main(String[] args){
        int pointCount;
        int pathCount;
        Node[] pathSet;
        Scanner cin = new Scanner(System.in);
        pointCount = cin.nextInt();
        pathCount = cin.nextInt();
        pathSet = new Node[pathCount];
        father = new int[pointCount+1];
        for(int i=0;i<pathCount;i++){
            pathSet[i] = new Node();
            int a = cin.nextInt();
            pathSet[i].a = a;
            int b = cin.nextInt();
            pathSet[i].b = b;
            int c = cin.nextInt();
            pathSet[i].val = c;
        }
        for (int i=1;i<=pointCount;i++){
            father[i] = i;
        }

        Arrays.sort(pathSet, new Comparator<Node>() {
            public int compare(Node o1, Node o2) {
                return o1.val < o2.val?-1:1;
            }
        });
        kruskar(pathSet);
    }
    //按排序后，从小到大（权值）的顺序链接节点
    public static void kruskar(Node[] pathSet){
        for(int i=0;i<pathSet.length;i++){
            unit(pathSet[i].a,pathSet[i].b,pathSet[i].val);
        }

        System.out.print(minPath);
    }
    //并查集的找祖先
    public static int find_root(int x){
        if(x != father[x]){
            father[x] = find_root(father[x]);
        }
        return father[x];
    }
    //并查集的集合合并
    public static void unit(int x,int y,int val){
        int rootX = find_root(x);
        int rootY = find_root(y);
        if(rootX!=rootY){
            father[rootX] = y;
            minPath+=val;
        }
    }

    public static class Node{
        int a;//节点a
        int b;//节点b
        int val;//a b距离

    }


}

```
