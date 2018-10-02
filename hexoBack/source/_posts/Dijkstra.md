---
title: Dijkstra
date: 2016-12-08 10:46:37
categories: algorithm
tags: algorithm
description: "Dijkstra解决的是单源最短路径问题，也就是图中某个点到其他任意点的最短路径，权值要求是正值。"
---
Dijkstra解决的是单源最短路径问题，也就是图中某个点到其他任意点的最短路径，权值要求是正值。
<!-- more -->
## 废话
今天看了Dijkstra算法，花了些时间，也总算是理解了，特此写下，方便以后回忆。感觉算法这东西，长时间不看，容易忘。当然网上这类博客也很多，其中也有些讲的很好。之所以自己写，是因为自己理解的思路，总归是有些自己的特点，如果每次都去看别人的，每次都要按别人的套路走一遍。而回过头看自己的话，估计浏览一遍就能回忆起来。我只能说写给自己以后温习用，因为自己理解不够深，文采又没有，不敢说写给广大人民群众看，当然，如果你看完后对你有点帮助那就再好不过了。废话不多说了，开始。
## 概述
Dijkstra解决的是单源最短路径问题，也就是图中某个点到其他任意点的最短路径，权值要求是正值。
## 讲故事
![此处输入图片的描述][1]
看这图是不是很简单？简单就对了，简单能说明问题的话，干嘛搞个很复杂的图。Dijkstra的思想有些像广搜，地毯式搜索，每次只找离自己最近的节点。我们可以这么想，比如我要找1到4的最短路。那么1就可以看成一个公司的boss，现在公司就他一个人。我们看到2和3与他相连，说明他认识2和3，2和3跟他的距离（权值）也就是跟他的亲密度。这时候，如果1想拉人入伙，我们想想他会找谁，肯定找3啊，因为3跟他关系好啊。那么也就是说，1节点首先找到离他最近的节点，那就是3.然后3入伙了，也就近了公司这个集合。1就说了，公司需要扩大规模，你快去发展下线（招人去吧）。然后就把招人的事交给3了，从图中我们可以看出，3只认识4了，所以就去找4，找到后跟1汇报，并说明了他跟4的关系，权值是10（看来关系有点远了）。这个时候，1想了，我不是还认识2吗，相比4，跟2的关系更好啊。于是，1又找到跟他关系最好的（距离最近），因为这个时候3已经加入了公司，所以不考虑了。所以就只能是2了。2加入后跟3做同样的事，去招人。巧了，2也认识4，并且跟4的关系看上去还不错奥。这个时候它再向1汇报，说自己找到了4，并且说明跟4的关系。这个时候注意，前面忘记交代，当3找到4并向1汇报的时候，1综合他跟3的关系，再加上3跟4的关系，已经能得出跟4的关系值了（距离），当时就是因为不太满意，感觉不可信，他才先就近找2.现在2也找到了4，并且关系又近了不少，1更新完跟4的关系值后，感觉还行，最后把4也收下了，这样，所有人都进入了公司，并且跟1的关系很明确了。
##故事&Dijkstra
读完故事，我们能得到这些信息
* boss(源点)每次都是找离他最近的人（节点）
* 找到节点后让他去招人（更新与最近节点相连的节点信息）
其实主要就是这两点了，不断循环往复，所有节点都会被找到，所有节点距源点的距离也变成了最短距离，因为源点每次派出去的都是据他最近的点。
##代码
```java
public class Dijkstra {
    //这里主要做初始化工作
    public static void main(String[] args){
        int count;//节点数
        int pathCount;//路径数
        Scanner scanner=new Scanner(System.in);
        count = scanner.nextInt();
        pathCount = scanner.nextInt();
        int[][] array = new int[count+1][count+1];
        for(int i=0;i<count+1;i++){
            for(int j=0;j<count+1;j++){
                if(i == j)array[i][j] = 0;
                else array[i][j] = Integer.MAX_VALUE;

            }
        }
        for(int i=0;i<pathCount;i++){
            int a = scanner.nextInt();
            int b = scanner.nextInt();
            int c = scanner.nextInt();
            array[a][b] = c;
        }
        System.out.print(dijkstra(array,1,5)+"");

    }
    //核心方法
    public static int dijkstra(int[][] array,int start,int end){
        int length = array.length;
        int[] dis = new int[length];//源点到各节点的距离
        boolean[] vis = new boolean[length];//是否已经加入公司了
        vis[start] = true;//开始只有boss在公司
        for(int i=1;i<length;i++){//boss先收集他认识的人
            dis[i] = array[start][i];
        }
        //找他最信任的人
        for(int i=1;i < length;i++){
            int current = 0;
            int min = Integer.MAX_VALUE;
            for(int j = 1;j<length;j++){
                if(!vis[j] && dis[j]<min){
                    min = dis[j];
                    current = j;
                }
            }
            
            vis[current] = true;//找到后这个人加入公司
            for(int v = 1;v<length;v++){//派去干活（招人）
                if(array[current][v]<Integer.MAX_VALUE){
                    dis[v] = dis[v] > (array[current][v]+dis[current])?(array[current][v]+dis[current]):dis[v];
                }
            }

        }
//        for(int i=1;i<length;i++){
//            System.out.print(dis[i]+" ");
//        }
        return dis[end];

    }
}
/*
6 9
1 2 7
1 3 9
1 6 14
2 3 10
3 6 2
6 5 9
2 4 15
3 4 11
4 5 6
 */
```
代码关键地方依照故事情景写了注释，应该是能理解了，感觉自己写的好通俗，失去了算法那种高大上的感觉。不过我不care，能让我最快最容易理解的方法就是好方法，当然如果能帮到正在看的人，那就是更好的方法了。


  [1]: http://ofy9dm2ii.bkt.clouddn.com/Dijkstra.png
