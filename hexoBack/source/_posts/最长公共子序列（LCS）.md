---
title: 最长公共子序列（LCS）
date: 2016-12-05 11:14:08
categories: algorithm
tags: algorithm
---
最长公共子序列问题，是典型的动态规划问题。个人认为，符合动态规划的需要满足两个基本的条件
* 目标问题可以划分成众多相同的子问题
* 子问题具有重叠性"
<!-- more -->
最长公共子序列问题，是典型的动态规划问题。个人认为，符合动态规划的需要满足两个基本的条件
* 目标问题可以划分成众多相同的子问题
* 子问题具有重叠性
划分子问题：当我们拿到一个问题的时候，不可能一眼看出答案，计算机也不行，都要经过复杂的计算。但是如果我们把问题规模变小一点，只看其中的一部分，可能就比较容易得出答案，而我们的出的问题一小部分的答案，对我们计算出此问题的答案是有帮助的。简单说就是子问题的最优解服务于目标问题。例如我们计算BDCABA和ABCBDAB两个串的最长公共子序列。我们一眼肯定看不出答案，但是如果问你B和ABCBDAB的公共子串，那么一眼就看出来了。然后再看BD和ABCBDAB的公共子串，因为我们知道了B和ABCBDAB的子串，所以只关心D和ABCBDAB的子串，然后加上B和ABCBDAB的结果。再往后就是同样的步骤循环往复。
重叠性：还是上面的例子，当我们计算B和ABCBDAB的公共子串时，要遍历ABCBDAB串，和B比较，判断是否有和B相同的。然后当我们计算BD和ABCBDAB的公共子串时，我们又要遍历ABCBDAB串，看是否有和B相等的，再判断是否有和D相等的。可以发现，找和B相等的这个步骤明显是重复的，我们在首个子问题已经判断过了。如果两个串非常长，这种重复操作就很耗时，又没有什么意义，dp就是解决这个问题的，他把子问题的最优解保存的表格中，当需要的时候，拿来用就是了，不用重复判断子问题的。
下面通过一张图（来自算法导论）分析下dp过程
![LCS][1]
从图中可以看出，它把子问题的结果都存在一张表中了。从左上到右下，问题规模不断扩大，当问题规模扩大的时候，都可以利用扩大之前的结果，在这个基础上继续计算。比如我们走到（x2,y2）这个子问题的时候，之前的子问题结果我们已经拿到了，而且我们知道，在之前的众多子问题中(x2,y1)这个子问题的最优解是符合我们的要求的，所以我们只需要判断当前节点，加上(x2,y1)这个子问题的结果，肯定就是目前的最优解了。这样直到最好，(Xmax,Ymax)就是我们要的最优解
```java
public static void main(String[] args) {
        String str1 = "BDCABA";
        String str2 = "ABCBDAB";
        System.out.print(lcs(str1, str2) + "");
    }

    public static int lcs(String str1, String str2) {
        int[][] dp = new int[str1.length()+1][str2.length()+1];
        for (int i = 0; i <= str1.length(); i++) {
            for (int j = 0; j <= str2.length(); j++) {
                dp[i][j] = 0;
            }
        }
        for (int i = 0; i < str1.length(); i++) {
            for (int j = 0; j < str2.length(); j++) {
                if (str1.charAt(i) == str2.charAt(j)) {
                    dp[i+1][j+1] = dp[i][j] + 1;
                } else {
                    dp[i+1][j+1] = Math.max(dp[i][j+1], dp[i+1][j]);
                }

            }
        }
        return dp[str1.length()][str2.length()];
    }
```


  [1]: http://ofy9dm2ii.bkt.clouddn.com/image/article/dp.png
