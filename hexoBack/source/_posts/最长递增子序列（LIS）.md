---
title: 最长递增子序列（LIS）
date: 2016-12-06 09:25:12
categories: algorithm
tags: algorithm
---
比如要找出ABDEC的最长子序列，我们可以先对这个串排序，排序后为ABCDE，此时找出两个串的最长公共子序列，即为ABDEC的最长递增子序列。[最长公共子序列][1]算法我在前面有写，可以参考一下。
<!-- more -->
## 排序+最长公共子序列
比如要找出ABDEC的最长子序列，我们可以先对这个串排序，排序后为ABCDE，此时找出两个串的最长公共子序列，即为ABDEC的最长递增子序列。[最长公共子序列][1]算法我在前面有写，可以参考一下。
## 直接dp
dp的两大规则，
* 目标问题可以划分为相同的众多子问题
* 子问题具有重叠性，子问题对目标问题的求解有帮助
看我们这题目，显然满足。首先，我们求ABDEC的LIS的时候，我们可以先求出A子串的LIS，在A的基础上再求出AB的LIS，这样从子问题出发，问题规模不断扩大，最终求出目标问题的解。
```java
public class LIS {
    public static void main(String[] args){
        String str = "ABDEC";
        System.out.print(lis(str)+"");
    }

    public static int lis(String str){
        int[] dp = new int[str.length()+1];
        for(int i=0;i<dp.length;i++){
            dp[i] = 0;
        }
        for(int i=0;i<str.length();i++){
            if(i == 0){
                dp[i] = 1;
            }else{
                for(int j = i-1;j>=0;j--){
                    if(str.charAt(i) > str.charAt(j)){
                        dp[i] = dp[j]+1;
                        break;
                    }
                }
            }
        }
        return dp[str.length()-1];
    }
}
```
简单分析下代码。首先创建一个辅助数组，用来保存ABDEC串的每个位置的最大递增子序列。例如dp[0] = 1.因为只有A一个的时候，长度是1.dp[1] = 2;1这个位置对应的就是串的B，也就是走到B时的LIS，很明显B比A大，所以在B位置就是2，依次分析，就得出答案。每走到一个位置，让这个位置的数据跟在他之前的每一位进行比较，当找到第一个比他小的时候，他的长度就是那个比他小的的长度+1；
## 最小末尾
这种方法肯定不是叫最小末尾，只是最小末尾是核心，就这么叫了。因为我们要找的是最长递增子序列，那么可以知道，子串的第二位肯定比第一位大，同样第三位肯定比第二位大。那么我们在第一位，肯定要放入一个最小的数字，如果放大了，那么可能存在比他小的，就没地方放了。就是在LIS的每一位，放入最小可能值。比如对1423这个序列。首先LIS的第一位肯定放1，然后第二位放4.当我们看到2的时候，2明显比4小啊，4放第二位就不合适了，那么就把4拿掉，2放第二位，看到3的时候，3比2大，所以放在第三位。又因为我们的辅助数组里按最小末尾放值，那么肯定是递增的，那么当我们拿到一个新的值，在辅助数组中替换某个最小末尾的时候，就可以用二分查找，节省时间
```java
public static int lis2(String str){
        int len = 1;
        char[] dp = new char[str.length()];
        for(int i=0;i<dp.length;i++){
            dp[i] = 0;
        }
        dp[len] = str.charAt(0);
        for(int i=1;i<str.length();i++){
            if(dp[len]< str.charAt(i)){
                dp[++len] = str.charAt(i);
            }else{
                int index = binary(i,dp,str);
                dp[index] = str.charAt(i);
            }
        }
        return len;
    }

    public static int binary(int i,char[] dp,String str){
        int left=0,right=dp.length-1;
        int mid ;
        while(left<right){
            mid = (left+right)/2;
            if(dp[mid]>str.charAt(i)) right = mid;
            else left = mid+1;
        }
        return left;
    }
```
此方法的时间复杂度降到了nlogn,而前面两个都是n^2




[1]: https://geekzw.github.io/2016/12/05/最长递增子序列（LIS）/
